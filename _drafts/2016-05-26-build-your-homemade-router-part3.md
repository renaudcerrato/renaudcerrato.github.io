---
published: false
layout: post
---

1-enable source in etc/apt/sources.list

UNAME=$(uname -r)
sudo apt-get build-dep linux-image-$UNAME
apt-get source linux-image-$UNAME
mkdir tmp
cp /boot/config-$UNAME  tmp/.config
cp /usr/src/linux-headers-${UNAME}/Module.symvers tmp/
cd linux-${UNAME%%-*}



