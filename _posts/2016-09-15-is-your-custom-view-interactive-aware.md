---
published: false
layout: post
title: Is your custom view interactive aware?
---
During the lifecycle of your Android [view](https://developer.android.com/reference/android/view/View.html), there are times where the user can't interact with it, and you ~~can~~ must take advantage of it to save some battery. Your custom views may need to go greener! At [Popsy](https://play.google.com/store/apps/details?id=com.mypopsy.android), we care about your battery.

## What's an interactive view?

A [view](https://developer.android.com/reference/android/view/View.html) can be said _interactive_ when she's ready to interact with the user or, in other words, when she's visible to the user. Thankfully, Android SDK developers gave us a bunch of protected callbacks to easily deal with that:


*  [`void View::onVisibilityChanged(View, int)`](https://developer.android.com/reference/android/view/View.html#onVisibilityChanged%28android.view.View, int%29)

 This is called when the visibility of the view (or an ancestor) of the view has changed (between GONE, INVISIBLE, and VISIBLE).
 
* [`void View::onWindowVisibilityChanged()`](https://developer.android.com/reference/android/view/View.html#onWindowVisibilityChanged%28int%29)

 Called when the window containing has change its visibility (between GONE, INVISIBLE, and VISIBLE).
 
* [`void View::onAttachedToWindow()`](https://developer.android.com/reference/android/view/View.html#onAttachedToWindow%28%29)

 This is called when the view is attached to a window. At this point it has a Surface and will start drawing.

* [`void View::onDetachedFromWindow()`](https://developer.android.com/reference/android/view/View.html#onDetachedToWindow%28%29) 

 This is called when the view is detached from a window. At this point it no longer has a surface for drawing.

The callbacks above will let you know if your view is ready to be drawn and/or visible within the view hierarchy, but you're still missing an important signal: is your device ready to interact with your user?

* [`Intent.ACTION_SCREEN_ON`](https://developer.android.com/reference/android/content/Intent.html#ACTION_SCREEN_ON)

 Broadcast Action sent when the device wakes up and becomes interactive. Below API 20, the name of this broadcast action refered to the power state of the screen. For API 20 and above, it is actually sent in response to changes in the overall [interactive state of the device](https://developer.android.com/reference/android/os/PowerManager.html#isInteractive%28%29).
 
* [`Intent.ACTION_SCREEN_OFF`](https://developer.android.com/reference/android/content/Intent.html#ACTION_SCREEN_OFF)

 Broadcast action sent when the device goes to sleep and becomes non-interactive. Below API 20, the name of this broadcast action refered to the power state of the screen. For API 20 and above, it is actually sent in response to changes in the overall [interactive state of the device](https://developer.android.com/reference/android/os/PowerManager.html#isInteractive%28%29).


By taking advantage of all of the above, you're now able to determine if your view is interactive.


## Ok, got it, and?

If your custom view is doing heavy things (like a loop animation for a loading spinner), relies on Android sensors (like a compass, or our own [DoorSignView](https://www.github.com/renaudcerrato/DoorSignView), or anything else required to be re-drawn periodically (like a [RelativeTimeTextView](https://github.com/curioustechizen/android-ago/blob/master/android-ago/src/com/github/curioustechizen/ago/RelativeTimeTextView.java)) then you ~~can~~ must take advantage of the signals above to save battery when your view is'nt interactive yet/anymore.

![](/static/img/spinner.gif){: .center-image }

![](/static/img/doorsign.gif){: .center-image }


## Great! I'm in!

You're in luck, I wrote a [small helper](https://gist.github.com/renaudcerrato/746e039700ac5eeaaea40808666e239f) which will handle everything for you - including registering to `ACTION_SCREEN_ON`/`ACTION_SCREEN_OFF` broadcasts:

{% gist 746e039700ac5eeaaea40808666e239f %}

To use it, just delegate the callbacks cited above, and implements `InteractiveViewHelper.Callback` to be notified:

```java
public class NastyCustomView extends View implements InteractiveViewHelper.Callback {

	private final InteractiveViewHelper mInteractiveViewHelper = 
    									new InteractiveViewHelper(this, this);

	...
    
	@Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        mInteractiveViewHelper.onAttachedToWindow();
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        mInteractiveViewHelper.onDetachedFromWindow();
    }

    @Override
    protected void onVisibilityChanged(@NonNull View changedView, int visibility) {
        super.onVisibilityChanged(changedView, visibility);
        mInteractiveViewHelper.onVisibilityChanged(changedView, visibility);
    }

    @Override
    protected void onWindowVisibilityChanged(int visibility) {
        super.onWindowVisibilityChanged(visibility);
        mInteractiveViewHelper.onWindowVisibilityChanged(visibility);
    }

    @Override
    public void onInteractivityChanged(boolean isInteractive) {
        enableNastyThings(isInteractive); // be greener!
    }
    
    ...
}
```











