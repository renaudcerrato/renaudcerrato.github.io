---
published: true
title: Easy video to GIF conversion
layout: post
tags: linux,script
---
If, like me, you're crawling GitHub for the latest awesome UI libraries - you know how frustrating it is when the author _"forgot"_ to insert a visual representation of its valuable work. [Some](https://github.com/evelyne24/ClockDrawableAnimation) are kind enough to insert a screenshot but are relying on a Youtube link - which add a lot of friction.

Since GitHub doesn't support video embeds, we have no choice but to use GIF - and [as a publisher](https://github.com/renaudcerrato) I sporadically needed a tool able to do that seamlessly without leaving the comfort of my keyboard.

Here come [my little handy script](https://gist.github.com/renaudcerrato/14af0fc855fda52e6587), which is a wrapper around `ffpmpeg` and [`imagemagick`](http://www.imagemagick.org):

```
$ video2gif.sh -h

Usage: video2gif.sh [OPTIONS...] <filename>
Convert a video to GIF.

Example: video2gif.sh -r24 -w320 -f3 video.mp4

 GIF format:

  -r <fps>,           set framerate
  -w <width>,         set width in pixel

 Color optimizations:

  -d <size>,          enable color dithering with <size> pattern
  -f <percent>        enable color fuzzing

 Output selection:

  -o <filename>,      output to <filename>

 Other options:
  -i,                 open folder after image extraction and before conversion
```

The `-r` switch will set the desired framerate while the `-w` switch will set the output width in pixels. From my own experience, a framerate between 12fps and 18fps will fit most cases. The `-d` and `-r` switches may shrink the resulting size further by adding some color dithering/fuzzing. 

You'll find below some outputs with varying dithering/fuzzing settings, at a fixed framerate of 15fps:

<table>
<tr>
<td><img src="/images/screen.gif"/></td>
<td><img src="/images/screend8f8.gif"/></td>
<td><img src="/images/screend16f8.gif"/></td>
</tr>
</table>

**Enjoy!**
