---
published: false
layout: post
---
During the lifecycle of your Android [view](https://developer.android.com/reference/android/view/View.html), there are times where the user can't interact with it, and you ~~can~~ must take advantage of it to save some battery. Your custom views may need to go greener! At [Popsy](https://play.google.com/store/apps/details?id=com.mypopsy.android), we care about your battery.

## What's an interactive view?

A [view](https://developer.android.com/reference/android/view/View.html) can be said _interactive_ when she's ready to interact with the user or, in other words, when she's visible to the user. 





*  [`onWindowVisibilityChanged()`](https://developer.android.com/reference/android/view/View.html#onWindowVisibilityChanged%28int%29)
* [`void onVisibilityChanged(View, int)`](https://developer.android.com/reference/android/view/View.html#onVisibilityChanged(android.view.View, int%29)
* [`onAttachedToWindow()`](https://developer.android.com/reference/android/view/View.html#onAttachedToWindow%28%29)
* [`onDetachedFromWindow()`](https://developer.android.com/reference/android/view/View.html#onDetachedToWindow%28%29)



