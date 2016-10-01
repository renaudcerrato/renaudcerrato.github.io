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

Based on [my previous setup]({% post_url 2016-05-23-build-your-homemade-router-part2 %}), I can draw a diagram of what we want to achieve. Our virtual SSID will run on a virtual `wlan0` interface, on its own `192.168.2.0/24` sub-network:

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

## Network interface configuration

According to the documentation found in `hostapd.conf`, there's a strong requirement between the MAC address of the physical interface, and the BSSID of the virtual interface(s):

```shell
hostapd will generate BSSID mask based on the BSSIDs that are
configured. hostapd will verify that dev_addr & MASK == dev_addr. If this is
not the case, the MAC address of the radio must be changed before starting
hostapd. If a BSSID is configured for
every secondary BSS, this limitation is not applied at hostapd and other
masks may be used if the driver supports them (e.g., swap the locally
administered bit)

BSSIDs are assigned in order to each BSS, unless an explicit BSSID is
specified using the 'bssid' parameter.
If an explicit BSSID is specified, it must be chosen such that it:
- results in a valid MASK that covers it and the dev_addr
- is not the same as the MAC address of the radio
- is not the same as any other explicitly specified BSSID
```

In order to fullfill the requirements above and let `hostapd` automatically assign the BSSID of our virtual interfaces without complaining, we'll change the MAC address of our wireless interface by forcing its four least significant bits to zero. That's enough room to allocate 16 virtual BSSID, which is more than necessary. 

First, Let's determine our current MAC address:

```shell
$ ethtool -P wlp5s0
Permanent address: 44:c3:06:00:03:eb
```

According to output the above, our current MAC address is `44:c3:06:00:03:eb`. By clearing its four least significant bits, and also setting [the U/L bit](https://en.wikipedia.org/wiki/MAC_address#Universal_vs._local), we obtain `46:c3:06:00:03:e0`.

Let's update our network interface configuration to change our MAC address:

```shell
$ cat /etc/network/interfaces
...
# Physical Wireless
auto wlp5s0
iface wlp5s0 inet manual
    pre-up ifconfig wlp5s0 hw ether 46:c3:06:00:03:e0
...
```


