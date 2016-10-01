---
published: true
title: Virtual SSID setup with hostapd
layout: post
tags: linux hardware
series: homemade-router
---
Wether you're willing to setup a guest access-point, or a dedicated wireless network for your VPN needs - you'll have to setup a virtual SSID at some point. This post will walk you through the required steps to achieve your goal using [hostapd](https://wiki.gentoo.org/wiki/Hostapd). 

First of all, this post is assuming you already setup your wireless card as an access-point. If this is not the case, head up to [my previous post]({% post_url 2016-05-23-build-your-homemade-router-part2 %}).


## Diagram

Based on [my previous setup]({% post_url 2016-05-23-build-your-homemade-router-part2 %}), we can draw a diagram of what we want to achieve. `wlp5s0` is our physical wireless interface, and our virtual SSID will run on a virtual `wlan0` interface, on its own `192.168.2.0/24` sub-network:

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

From the output above, we see that the chipset supports up to 8 AP **on a single channel**. That mean our virtual SSIDs will run on the same wireless channel as the _real_ one.

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

In order to fullfill the requirements above and let `hostapd` automatically assign the BSSID of our virtual interfaces without complaining, we'll change the MAC address of our physical wireless interface by forcing its four least significant bits to zero. That's enough room to allocate 15 virtual BSSID, which is more than necessary. 

First, let's determine our current MAC address:

```shell
$ ethtool -P wlp5s0
Permanent address: 44:c3:06:00:03:eb
```

According to the output the above, by clearing the four least significant bits, and also setting [the U/L bit](https://en.wikipedia.org/wiki/MAC_address#Universal_vs._local) for sanity, our new MAC address would be `46:c3:06:00:03:e0`.

Now, let's update our network interface configuration to update our MAC address right before the interface is brought up, and also declare our virtual interface according to our diagram:

```shell
$ cat /etc/network/interfaces
...
# Physical Wireless
auto wlp5s0
iface wlp5s0 inet manual
    pre-up ifconfig wlp5s0 hw ether 46:c3:06:00:03:e0

# Virtual Wireless
allow-hotplug wlan0
iface wlan0 inet static
    address 192.168.2.1
    network 192.168.2.0
    netmask 255.255.255.0
    broadcast 192.168.2.255
    post-up /usr/sbin/dnsmasq \
		--pid-file=/var/run/dnsmasq-wlan0.pid \
		--conf-file=/dev/null \
		--interface=wlan0 --except-interface=lo \
		--bind-interfaces \
		--dhcp-range=192.168.2.10,192.168.2.150,24h
    post-down cat /var/run/dnsmasq-wlan0.pid | xargs kill
...
```

That's all. As we may see above, I'm using `dnsmasq` for our DHCP and DNS needs - feel free to use your prefered weapons. Please note that `allow-hotplug` is required on our virtual interface to properly work.

## Access point configuration

Now, the easiest part: we'll add a virtual SSID to our current `hostapd` configuration. Simply append, **at the bottom** of your existing `hostapd.conf`, the desired configuration:

```shell
$ cat /etc/hostapd/hostapd.conf
...
### Virtual SSID(s) ###
bss=wlan0
ssid=MyVirtualSSID
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase=you_cant_guess
```

In the example above, I used WPA2 but most of the options are available (apart from radio interface specific items, like channel). We could add more virtual SSID by simply appending more configurations - according we declared and configured more virtual interface.

Now, simply reboot, and you should be able to see your new SSID, along with your new wireless interface (notice the MAC address):

```shell
$ ifconfig wlan0
wlan0     Link encap:Ethernet  HWaddr 46:c3:06:00:03:e1  
          inet addr:192.168.2.1  Bcast:192.168.2.255  Mask:255.255.255.0
          inet6 addr: fe80::c3:6ff:fe00:3e1/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:557512 errors:0 dropped:0 overruns:0 frame:0
          TX packets:852606 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:73841867 (73.8 MB)  TX bytes:1015056727 (1.0 GB)
```

**That's all folks!**