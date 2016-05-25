---
published: true
title: Build your homemade router (part 2)
layout: post
---

This post is the second part of the series "Build your homemade router". The [first part](https://renaudcerrato.github.io/2016/05/21/build-your-homemade-router-part1/) covered the material installation.

That second part will cover the configuration of your freshly installed machine and, before going further, we'll need to draw what we want to achieve. Let's see what are the name of the every interface:

```shell
$ ifconfig
enp1s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx           
          ...
enp2s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx  
          ...
wlp5s0    Link encap:Ethernet  HWaddr xx:xx:xx:xx:xx:xx
          ...         
```

The motherboard have 2 built-in NIC named `enp1s0` and `enp2s0`. The mini-PCIe WiFi card is showing as `wlp5s0`, and support AP mode as expected:

```shell
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

From the informations above, we can draw our diagram: the first NIC will serve as our WAN port, while the second one will be bridged to our wireless network:


![](/static/img/network-diagram.jpg)



## Network

We'll make use of [dnsmasq](http://manpages.ubuntu.com/manpages/xenial/man8/dnsmasq.8.html) as our DHCP/DNS server:

```shell
$ sudo apt-get install dnsmasq bridge-utils
```

We'll need to edit our [network interface configuration](http://manpages.ubuntu.com/manpages/xenial/man5/interfaces.5.html) to match our diagram: you'll find below a _preliminary_ configuration including a minimal `dnsmasq` setup:

```shell
$ cat /etc/network/interfaces
# Loopback
auto lo
iface lo inet loopback

# WAN interface
auto enp1s0
iface enp1s0 inet dhcp

# Bridge (LAN)
auto br0 
iface br0 inet static
    address 192.168.1.1
    network 192.168.1.0
    netmask 255.255.255.0
    broadcast 192.168.1.255 
    bridge_ports enp2s0 wlp5s0
    post-up /usr/sbin/dnsmasq \
              --pid-file=/var/run/dnsmasq-br0.pid \
              --conf-file=/dev/null \
              --interface=br0 --except-interface=lo \
              --dhcp-range=192.168.1.10,192.168.1.150,24h
    pre-down cat /var/run/dnsmasq-br0.pid | xargs kill
```

As you may see, `dnsmasq` will be started as soon as the bridge is up thanks to the `post-up` section. Its configuration is done through command line arguments only (`--conf-file=/dev/null`) and the process will be shutdown when the interface go down. 

> The documentation of each arguments can be found on the [man page](http://manpages.ubuntu.com/manpages/xenial/man8/dnsmasq.8.html).

You can now restart your networking (`sudo service networking restart`) or simply reboot the router to check if your network configuration is properly setup. 

However, please note that while you should be able to get DHCP leases from `enp2s0` at this point, **you won't be able to connect wirelessly** (more on this later), **nor able to connect to internet** (see below).

## Routing

At this point, we need to tell our machine to route packets between our LAN (`enp2s0`) and WAN interface (`enp1s0`), and enable [masquerading](http://www.billauer.co.il/ipmasq-html.html) on it.

First of all, we need to enable packet forwarding:

```shell
$ sudo sysctl -w net.ipv4.ip_forward=1
$ echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```

> The last command above will ensure that the configuration will survive to the next reboot.

Like me, if you read tons of documentations/tutorials/articles about `iptables` and still can't understand how an human brain could ever learn to write [iptables](https://help.ubuntu.com/community/IptablesHowTo) statements, you're right! Being required to write iptables statements today is like being required to write programs in assembly. Hopefully, the guys at [FireHol](https://firehol.org/) put a lot of efforts adding the required level of abstration to that:

```shell
$ sudo apt-get install firehol
```

Thanks to `firehol`, our routing configuration could be as simple as:

```shell
$ cat /etc/firehol/firehol.conf 
version 6

# Accept all client traffic on wan interface
interface enp1s0 wan
        client all accept

# Accept all traffic on lan interface
interface br0 lan
        server all accept
        client all accept

# Route packets between both interface
router lan2wan inface br0 outface enp1s0
        masquerade
        route all accept
```

You can test the above setup by starting `firehol` manually (`sudo firehol start`) and by connecting a laptop to the available NIC port: **you should now be able to browse the internet.**

As a last point, **do not forget** to edit `/etc/default/firehol` to enable autostart:

```shell
$ cat /etc/default/firehold
...
# To enable firehol at startup set START_FIREHOL=YES
START_FIREHOL=YES
```

> I won't go into details about the whole `firehol` syntax, the configuration should be almost self-explanatory but I'd recommend to give a look at their [documentation](https://firehol.org/documentation/) for a more complex setup. Moreover, if you're really curious about what `firehol` did to our iptables, just drop `sudo firehol status` on the command line.


## Access Point

With no surprise, we'll use [hostapd](https://wiki.gentoo.org/wiki/Hostapd) to manage our access-point:

```shell
$ sudo apt-get install hostapd
```

Before going further, I'd recommend to uninstall `NetworkManager` since we don't need it anymore and because [of a known bug](https://bugs.launchpad.net/ubuntu/+source/wpa/+bug/1289047):

```
$ sudo nmcli radio wifi off && sudo rfkill unblock wlp5s0
$ sudo apt-get remove network-manager
```

You'll find below a curated (but working) 802.11n/2.4Ghz/WPA2-AES configuration file:

```shell
$ cat /etc/hostapd/hostapd-test.conf 
#### Interface configuration ####
interface=wlp5s0
bridge=br0
driver=nl80211

##### IEEE 802.11 related configuration #####
ssid=MyRouter
hw_mode=g
channel=1
auth_algs=1
wmm_enabled=1

##### IEEE 802.11n related configuration #####
ieee80211n=1

##### WPA/IEEE 802.11i configuration #####
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_passphrase=YouCantGuess
```

You can easily test the configuration above by running the following command:

```shell
$ sudo hostapd /etc/hostapd/hostapd-test.conf
```

> You'll find the [documentation](http://w1.fi/gitweb/gitweb.cgi?p=hostap.git;a=blob_plain;f=hostapd/hostapd.conf) of every available options into the `/usr/share/doc/hostapd/examples/` directory. 

If everything goes well, **you should now be able to connect wirelessly**. However, **do not forget** to edit our interface configuration to start `hostapd` right before the interface goes up. 

**Here's our final `/etc/network/interfaces`**:

```shell
$ cat /etc/network/interfaces
# Loopback
auto lo
iface lo inet loopback

# WAN interface
auto enp1s0
iface enp1s0 inet dhcp

# Bridge (LAN)
auto br0 
iface br0 inet static
    address 192.168.1.1
    network 192.168.1.0
    netmask 255.255.255.0
    broadcast 192.168.1.255 
    bridge_ports enp2s0 wlp5s0
    pre-up /usr/sbin/hostapd \
              -P /var/run/hostapd-br0.pid \
              -B /etc/hostapd/hostapd-test.conf
    post-up /usr/sbin/dnsmasq \
              --pid-file=/var/run/dnsmasq-br0.pid \
              --conf-file=/dev/null \
              --interface=br0 --except-interface=lo \
              --dhcp-range=192.168.1.10,192.168.1.150,24h               
    pre-down cat /var/run/dnsmasq-br0.pid | xargs kill
    pre-down cat /var/run/hostapd-br0.pid | xargs kill

```


Stay tuned for the next part to enable a 802.11ac (5Ghz) access point!
