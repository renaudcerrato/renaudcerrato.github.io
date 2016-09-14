---
published: false
layout: post
---
During the lifecycle of your Android [view](https://developer.android.com/reference/android/view/View.html), there are times where the user can't interact with it, and you ~~can~~ must take advantage of it to save some battery. Your custom views may need to go greener! At [Popsy](https://play.google.com/store/apps/details?id=com.mypopsy.android), we care about your battery.

## What's an interactive view?

A [view](https://developer.android.com/reference/android/view/View.html) can be said _interactive_ when she's ready to interact with the user or, in other words, when she's visible to the user. Thankfully, Android SDK developers gave us a bunch of protected methods to easily deal with that:


*  [`void onVisibilityChanged(View, int)`](https://developer.android.com/reference/android/view/View.html#onVisibilityChanged%28android.view.View, int%29)

 This is called when the visibility of the view or an ancestor of the view has changed (between GONE, INVISIBLE, and VISIBLE).
 
* [`onWindowVisibilityChanged()`](https://developer.android.com/reference/android/view/View.html#onWindowVisibilityChanged%28int%29)

 Called when the window containing has change its visibility (between GONE, INVISIBLE, and VISIBLE).
 
* [`onAttachedToWindow()`](https://developer.android.com/reference/android/view/View.html#onAttachedToWindow%28%29)

 This is called when the view is attached to a window. At this point it has a Surface and will start drawing.

* [`onDetachedFromWindow()`](https://developer.android.com/reference/android/view/View.html#onDetachedToWindow%28%29) 

 This is called when the view is detached from a window. At this point it no longer has a surface for drawing.

## So?







