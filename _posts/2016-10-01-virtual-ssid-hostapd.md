---
published: false
title: Virtual SSID setup with hostapd
layout: post
tags: linux hardware
series: homemade-router
---
Wether you're willing to setup a guest access-point, or a dedicated wireless network for your VPN needs - you'll have to setup a virtual SSID at some point. This post will walk you through the required steps to achieve your goal using [hostapd](https://wiki.gentoo.org/wiki/Hostapd). 

First of all, this post is assuming you already setup your wireless card as an access-point. If this is not the case, head up to [my previous post]({% post_url 2016-05-23-build-your-homemade-router-part2 %}).

![](images/router-dual-ssid.png) {: .center-image }
