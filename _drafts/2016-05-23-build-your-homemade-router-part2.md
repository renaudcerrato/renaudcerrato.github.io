---
published: false
title: Build your homemade router (part 2)
layout: post
---

This post is the second part of the series "Build your homemade router". The [first part](https://renaudcerrato.github.io/2016/05/21/build-your-homemade-router-part1/) covered the material installation.

That second part will cover the configuration of your freshly installed machine and, before going further, we'll need draw what we want to achieve. First of all, let's see what are the name of the interfaces:

```bash
$ ifconfig
enp1s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx           
          ...
enp2s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx  
          ...
wlp5s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx
          ...         
```

The motherboard have 2 built-in LAN, named `enp1s0` and `enp2s0`. The mini-PCIe WiFi card is showing up as `wlp5s0`, and support AP mode as expected:

```
$ iw list
...
Supported interface modes:
                 * managed
                 * AP
                 * AP/VLAN
                 * monitor
                 * mesh point
```


## Diagram

From the informations above, we can finally draw our diagram: the first NIC will serve as our WAN port, while the second one will be bridged to our wireless network:


![](/static/img/network-diagram.jpg)



## Configuration


```
sudo apt-get install hostapd
```

Conflicting NetworkManager:
http://askubuntu.com/questions/472794/hostapd-error-nl80211-could-not-configure-driver-mode

Solution:

```
sudo apt-get remove network-manager
```

Dnsmasq + bridge: unknown network interface
https://bugs.launchpad.net/ubuntu/+source/dnsmasq/+bug/1531184


ath10k patches for 5Ghz:
https://dev.openwrt.org/browser/trunk/package/kernel/mac80211/patches
