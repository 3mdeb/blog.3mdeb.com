---
layout: post
title: "OpenWRT in QEMU"
date: 2014-08-10 19:04:17 +0200
comments: true
categories: embedded, openwrt
---

In below article I would like to go through process of configuring, building
and running [OpenWRT](https://openwrt.org/) in
[QEMU](http://wiki.qemu.org/Main_Page) using `realview-eb-mpcore` machine as a
target.

## OpenWRT

We will use latest OpenWRT version from git repository:

```
git clone git://git.openwrt.org/openwrt.git
```

Enter configuration by `make menuconfig` and choose below options:
```
Target System -> ARM Ltd. Realview board (qemu)
```
Compile using simple:
```
make
```
