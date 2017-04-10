---
layout: post
title:  "Runtime Subscriptions in Otto"
date:   2016-07-08 06:51:59
author: abhishek
categories: Tech, Android
comments: true
---
One should strive for loosely coupled design in software development. [Otto](http://square.github.io/otto/) is great library for Android which keep objects loosely coupled but still allow them to communicate efficiently.

Otto provides very simple mechanism for event publishing/subscribing which is based on Java annotations like `@Subscribe` and `@Produce`. However, while events can be posted on Runtime using `post()` method, subscriptions are compile time only.
In my recent project most of my objects/views were getting generated at runtime and I needed something which allows me to setup communication at runtime. `Otto` was already included in my project and then I posted [this](http://stackoverflow.com/questions/37839074/otto-event-bus-runtime-subscription-for-events) question on stackoverflow for help. I got two interesting solutions there.
Lets discuss them one by one

####Solution 1 - Use different model objects each subscribing for a different event
Clever! But, I see following problems with this model
<ol>
<li>What if its not known that which events this object should subscribe to at object creation time. </li>
<li>What if subscription rules state that this object should subscribe to multiple events ? </li>
<li>Creating new variant for same model based on what message its receiving ? <i>It sounds really messy !</i></li>
</ol>

####Solution 2 - Have a generic event and a event type in that then send this secret to everyone !! (<i>SSsshhhhhh...</i>)
At first glance it seems like a neater solution and might be good enough for some situations, but
<ol>
<li>Every message is sent to every object. First I smell performance problems, second I have to add extra code to ignore non-relevant messages in every object.</li>
</ol>
### My Approach
I propose following approach which is a improvement above Solution1 and Solution2. Instead of having different Model classes for different event types lets have different `EventDelegate` for different events. These event delegate will just receive and event and pass them on to models(or views or anything else which is interested party for that matter).

A typical parent event delegate should look like this
{% highlight java %}
public abstract class OttoEventDelegate {
    private OttoEventListener ottoEventListener;

    public OttoEventDelegate(OttoEventListener ottoEventListener) {
        this.ottoEventListener = ottoEventListener;
    }

    public void register() {
        BaseApplication.getInstance().getBus().register(this);
    }

    public void unregister() {
        BaseApplication.getInstance().getBus().unregister(this);
    }

    public OttoEventListener getOttoEventListener() {
        return ottoEventListener;
    }

    public void setOttoEventListener(OttoEventListener ottoEventListener) {
        this.ottoEventListener = ottoEventListener;
    }
}
{% endhighlight %}

Now that I mentioned that each event should have its own delegate here is what child class look like
{% highlight java %}
public class Event1Delegate extends OttoEventDelegate {

    public Event1Delegate(OttoEventListener ottoEventListener) {
        super(ottoEventListener);
    }

    @Subscribe
    public void onOttoEvent(Event1 event) {
        getOttoEventListener().onEventReceived(event);
    }
}
{% endhighlight %}

`OttoEventListener` is an interface which receiving party should implement in order to receive required event.

{% highlight java %}
public interface OttoEventListener {
    void onEventReceived(OttoEvent event);
}
{% endhighlight %}

Now each model can create as many different delegate objects on runtime and subscribe to relevant events and get notified of them like below
{% highlight java %}
public void runtimeSubscribe() {
        mDelegate = new Event1Delegate(new OttoEventListener() {
            @Override
            public void onEventReceived(OttoEvent event) {
                if(event instanceof Event1) {
                    Timber.d("Receiving Event in Model1: %s", ((Event1) event).getString());
                }
            }
        });

        mDelegate.register();
    }
{% endhighlight %}

also they should be able to unsubscribe in runtime at will

{% highlight java %}
public void unsubscribe() {
        mDelegate.unregister();
}
{% endhighlight %}

Note that above functions can be called at runtime based on any rules that might apply. These rules can come from API, or they can be result of some other runtime operations.

While there is some amount of work, it provides a flexible and scalable way of subscribing Otto events at runtime.
[This Demo App](https://github.com/abhishekBansal/OttoRuntime) on `github` demonstrates this mechanism. There are three different models, each of these model subscribe to events at runtime from `MainActivity` based on certain rules(I have kept things extremely simple here but these rules can be as complicated).
There are 3 buttons in `MainActivity`, each button emits an event which is captured by subscribed model and reflects it in logs. `MultiEventModel` shows that how this framework can even be used for subscribing to multiple events at runtime with little help from `Reflection`.

Here is a class diagram from same repository to summarize everything
<img src="/assets/images/otto-class-diagram.png" alt="Class Diagram - Otto Runtime Subscriptions" style="width: 900px; margin: auto;"/>

Here I have kept one `OttoEventDelegate` object in each `Model` for simplicity, but, A list can kept and handled as per use case. As many delegate objects can be added in that list enabling `Model` to subscribe to as many events at runtime.

Happy Coding !!

{% if page.comments %}
{% include disqus.html %}
{% endif %}
