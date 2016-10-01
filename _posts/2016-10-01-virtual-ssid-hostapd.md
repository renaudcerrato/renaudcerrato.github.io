---
published: false
title: Virtual SSID setup with hostapd
layout: post
tags: linux hardware
series: homemade-router
---
Wether you're willing to setup a guest access-point, or a dedicated wireless network for your VPN needs - you'll have to setup a virtual SSID at some point. This post will walk you through the required steps to achieve your goal using [hostapd](https://wiki.gentoo.org/wiki/Hostapd). 

First of all, this post is assuming you already setup your wireless card as an access-point. If this is not the case, head up to [my previous post]({% post_url 2016-05-23-build-your-homemade-router-part2 %}).


## Diagram

Based on [my previous setup]({% post_url 2016-05-23-build-your-homemade-router-part2 %}), we can draw what we want to achieve. Our virtual SSID will run on a virtual `wlan0` interface, on its own `192.168.2.0/24` sub-network:

![](/images/router-dual-ssid.png){: .center-image }


## Preliminary 

First of all, let's check that our wireless card supports virtual SSIDs:

```shell
$ iw list
...
	valid interface combinations:
    	* #{ AP, mesh point } <= 8,
        total <= 8, #channels <= 1, STA/AP BI must match
...
```

From the output above, we see that the chipset supports up to 8 virtual SSIDs - **on a single channel**. That mean our virtual SSID will run on the same wireless channel as the _real_ one.

## Setup

TODO
