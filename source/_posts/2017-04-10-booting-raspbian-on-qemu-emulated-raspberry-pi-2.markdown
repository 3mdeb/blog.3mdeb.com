---
author: Piotr Król
layout: post
title: "Booting Raspbian on QEMU emulated Raspberry Pi 2"
date: 2017-04-10 16:56:11 +0200
comments: true
categories: raspberrypi raspbian linux
---

Huge success of my [first post](2015/12/30/emulate-rapberry-pi-2-in-qemu/)
published almost 1.5 year ago makes me think I have to revisit previous article
and describe most recent development in area of Raspberry Pi 2 emulation in
QEMU.

I started research with what really happen on qemu-devel mailing lists


### QEMU compilation

```
git submodule update --init dtc
git submodule update --init pixman
./configure --target-list=arm-softmmu
make -j$(nproc)
```
