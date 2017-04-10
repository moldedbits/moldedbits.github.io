---
layout: post
title:  "Correct PopupWindow initialization"
date:   2016-09-29 6:50:00
author: anuj
categories: Technical Android
---
In a recent application, I needed to use the PopupWindow on a screen. Simple enough. While development, I used the Nexus 6P running Android Nougat for testing and everything worked fine and dandy. The QA, however, reported that the Popup was not visible on Android 5.0 or 4.4.2 . After much trial and error, I found the trouble making code.

Here's how I initialized the popup initially,

```Java
mPopup = new PopupWindow(getContext());
View view = inflater.inflate(R.layout.bar_graph_popup, null, false);
mPopup.setContentView(view);

// ... update view, calculate x and y for popup

mPopup.showAtLocation(v, Gravity.NO_GRAVITY, x, y);
```

This worked on Nougat, but on the older versions, there would be no popup on the screen, and view.getWidth() would return 0.

I then checked out example codes, and every example on the internet used the other constructor for PopupWindow

```Java
mPopup = new PopupWindow(view, ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);

// ... update view, calculate x and y for popup

mPopup.showAtLocation(v, Gravity.NO_GRAVITY, x, y);
```

I switched the constructor, and voila, everything started to work as expected. This was interesting, so I looked inside the two constructors to see what was going on.

```Java
public PopupWindow(View contentView, int width, int height, boolean focusable) {
    if (contentView != null) {
        mContext = contentView.getContext();
        mWindowManager = (WindowManager) mContext.getSystemService(Context.WINDOW_SERVICE);
    }

    setContentView(contentView);
    setWidth(width);
    setHeight(height);
    setFocusable(focusable);
}
```

While setting the context and the contentView, it was also setting the width and height. So I made some changes to my original code,

```Java
mPopup = new PopupWindow(getContext());
View view = inflater.inflate(R.layout.bar_graph_popup, null, false);
mPopup.setContentView(view);
mPopup.setWidth(ViewGroup.LayoutParams.WRAP_CONTENT);
mPopup.setHeight(ViewGroup.LayoutParams.WRAP_CONTENT);

// ... update view, calculate x and y for popup

mPopup.showAtLocation(v, Gravity.NO_GRAVITY, x, y);
```

And everything worked as intended.

Why this worked on Nougat and not below is still a mystery to me, hopefully I will be able to investigate that some day.

Happy coding!

The moldedbits Team
