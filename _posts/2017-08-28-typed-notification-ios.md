---
layout: post
title:  "Typed Notification iOS"
date:   2017-08-28 11:11:11
author: amit
categories: Technical iOS
---
Dealing with [NotificationCenter](https://developer.apple.com/documentation/notificationcenter) is daily task for all iOS developers. Either it's a system notification such as [KeyboardShow/KeyboardHide](https://developer.apple.com/library/content/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/KeyboardManagement/KeyboardManagement.html) or CustomNotification to post some information app wide, we all have used Notifications.

### Yeah, So. Do you have something interesting to say?

Well, answer to that question is, Yes. Today We are gonna discuss notification programming in such a way, that it will become as easy as eating an apple pie. (I don't think anyone can deny it, eating an apple pie is just so satisfying, obviously if you love apple pie in first place ;p )


### Old way of handling notification

In Swift we focus on writing all the APIs in strongly typed manner, then why not same with Notifications? Let's see an example of default way of doing notification handling.

```Swift
// Registering system notification for keyboardShow and keyboardHide
func registerForKeyboardNotifications() {
        NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillShow(_:)), name: NSNotification.Name.UIKeyboardWillShow, object: nil)
        NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillHide(_:)), name: NSNotification.Name.UIKeyboardWillHide, object: nil)
}

// Action on keyboardShow notification receive
func keyboardWillShow(_ notification: Notification) {
    let keyboardSize = (notification.userInfo?[UIKeyboardFrameBeginUserInfoKey] as? NSValue)?.cgRectValue
    let keyboardDuration = notification.userInfo?[UIKeyboardAnimationDurationUserInfoKey] as? Double
      //do something with these value
    }
}

// Action on keyboardShow notification receive
func keyboardWillHide(_ notification: Notification) {
  let keyboardDuration = notification.userInfo?[UIKeyboardAnimationDurationUserInfoKey] as? Double
}

//Removing notifies on viewController deinit
func deregisterFromKeyboardNotifications(){
    NotificationCenter.default.removeObserver(self, name: NSNotification.Name.UIKeyboardWillShow, object: nil)
    NotificationCenter.default.removeObserver(self, name: NSNotification.Name.UIKeyboardWillHide, object: nil)
}
```

We write this same code again and again or we create a `BaseViewController` and have these defined or maybe a `UIViewController` extension. But all of these are still not efficient as you have to dig into `notification.userInfo` dictionary to get relevant information.

### Typed Notificaion

We want to define notifications in such a way, where we can definitively get the data from our notification observer (strongly typed), not a userInfo dictionary. This way we will enforce swift type system to help us writing type safe and bug free code. In this process we will also redefine how we observe our notification to make our code `DRY`.

To create typed notification we can first define an `NotificationDescriptor` struct.

```Swift
struct NotificationDescriptor<A> {
    //Name of the notification
    let name: Notification.Name

    //Method to convert notification.userInfo to desired type
    let convert: (Notification) -> A
}
```

With help of Swift's Generic type we can use above Struct for all type of notification and describe our notifications using this.

Next we can extend `NotificationCenter` to provide us an convenience method over default `addObserver` method.

```Swift
extension NotificationCenter {
    func addObserver<A>(forDescriptor d: NotificationDescriptor<A>, using block: @escaping (A) -> ()) {
        addObserver(forName: d.name, object: nil, queue: nil, using: { notification in
            block(d.convert(notification))
        })
    }
}
```

But there is one thing missing in above implementation. We have added the observer but we are ignoring the token received from `addObserver` method. Without this token we won't be able to deregister the notification. For this purpose we can define a `Token` class.

```Swift
class Token {
    let center: NotificationCenter
    let token: NSObjectProtocol

    init(token: NSObjectProtocol, center: NotificationCenter) {
        self.token = token
        self.center = center
    }

    deinit {
        center.removeObserver(token)
    }
}
```

And redefine out `addObserver` method this way:-

```Swift
extension NotificationCenter {
    func addObserver<A>(forDescriptor d: NotificationDescriptor<A>, using block: @escaping (A) -> ()) -> Token {
        let t = addObserver(forName: d.name, object: nil, queue: nil, using: { notification in
            block(d.convert(notification))
        })
        return Token(token: t, center: self)
    }
}
```

We can store the `Token` returned by above function as a ViewController property, and as soon as ViewController's `deinit` gets called, `Token` will also gets destroyed and notification gets deregistered.

### How to use it

We will define KeyboardShow and Keyboard hide notification using our `NotificationDescriptor`.

```Swift
let keyboardShowNotification = NotificationDescriptor<A>(name: Notification.Name.UIKeyboardWillShow, convert: { notification in
  // Here we can parse the userInfo into our desired type
})

let keyboardHiderNotification = NotificationDescriptor<KeyboardPayload>(name: Notification.Name.UIKeyboardWillHide, convert: convert: { notification in
  // Here we can parse the userInfo into our desired type
})

```

Let's define a struct with info we require from keyboard notification using [KeyboardNotificationKeys](https://developer.apple.com/documentation/uikit/uiwindow/keyboard_notification_user_info_keys).

```Swift
struct KeyboardData {
    let beginFrame: CGRect
    let endFrame: CGRect
    let curve: UIViewAnimationCurve? //If we want to use it to synchronize our animation
    let duration: TimeInterval
}
```

We can also define an initializer to our `KeyboardData` which take notification and instantiate `KeyboardData`. This can be used as our convert function and we don't have to redefine convert for KeyboardShow and KeyboardHide.

This is also the most important part of our `Type-Safe Notification`. As now we will have strictly defined data in our application and all the parsing logic is separated out for easy debugging and testing.

```Swift
extension KeyboardData {
    init(note: Notification) {
        guard let userInfo = note.userInfo else {
            self.beginFrame = CGRect.zero
            self.endFrame = CGRect.zero
            self.curve = nil
            self.duration = 0

            return
        }

        self.beginFrame = userInfo[UIKeyboardFrameBeginUserInfoKey] as! CGRect
        self.endFrame = userInfo[UIKeyboardFrameEndUserInfoKey] as! CGRect
        self.curve = UIViewAnimationCurve(rawValue: (userInfo[UIKeyboardAnimationCurveUserInfoKey] as? Int) ?? 0)
        self.duration = userInfo[UIKeyboardAnimationDurationUserInfoKey] as! TimeInterval
    }
}
```

`KeboardShowDescriptor` and `KeyboardHideDescriptor` will become

```Swift
struct SystemNotification {
    static let keyboardShowNotification = NotificationDescriptor<KeyboardData>(name: Notification.Name.UIKeyboardWillShow, convert: KeyboardData.init)
    static let keyboardHiderNotification = NotificationDescriptor<KeyboardData>(name: Notification.Name.UIKeyboardWillHide, convert: KeyboardData.init)
}
```

We have also encapsulated these properties in `SystemNotification` struct, so that our global namespace doesn't get polluted with all different types of notification descriptors.

### UIViewController usage

As we have these properties already define, using in a UIViewController will become very simple and we don't have to track the notification for deregister.

```Swift
private var keyboardShowToken: Token?
private var keyboardHideToken: Token?

override func viewDidLoad() {
    super.viewDidLoad()

    registerForKeyboardNotifications()
}

func registerForKeyboardNotifications() {
        keyboardShowToken = NotificationCenter.default.addObserver(descriptor: SystemNotification.keyboardShowNotification) { keyboardData in
          //Custom logic related to view only
        }

        keyboardHideToken = NotificationCenter.default.addObserver(descriptor: SystemNotification.keyboardHideNotification) { keyboardData in
            //Custom logic related to view only
        }
    }
```

This way we are also freeing up our UIViewController from other logic and only allowing it to handle views. This is also highly reusable as all the repetitive code is moved to a single place and this makes our code `DRY`.

For MVVM architecture above implementation fits perfect, as in MVVM a viewController should only handle view related logic only.

### Inspiration

Above pattern is inspired from [SwiftTalk Objc.io](https://talk.objc.io/episodes/S01E27-typed-notifications-part-1). I would highly recommend watching this episode. They also define a different pattern using Protocol for handling the Notification. But for our use case we are fine with above implementation, much simpler and easy to use.

Happy coding!

The moldedbits Team
