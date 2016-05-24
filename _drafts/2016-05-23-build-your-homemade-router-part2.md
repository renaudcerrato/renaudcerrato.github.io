---
published: false
---

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
