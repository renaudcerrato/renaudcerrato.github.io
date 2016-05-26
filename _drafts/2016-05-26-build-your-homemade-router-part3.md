---
published: false
layout: post
---

1-enable source in etc/apt/sources.list

UNAME=$(uname -r)
sudo apt-get build-dep linux-image-$UNAME
apt-get source linux-image-$UNAME
mkdir ~/tmp
cp /boot/config-$UNAME  ~/tmp/.config && cp /usr/src/linux-headers-${UNAME}/Module.symvers ~/tmp/
cd linux-${UNAME%%-*}
make EXTRAVERSION=-${UNAME#*-} O=~/tmp oldconfig && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp prepare && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp modules_prepare && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp SUBDIRS=scripts/mod && \
make EXTRAVERSION=-${UNAME#*-} O=~/tmp modules SUBDIRS=drivers/net/wireless/ath

sudo depmod -a



