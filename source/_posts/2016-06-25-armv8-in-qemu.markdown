---
layout: post
title: "ARMv8 in QEMU"
date: 2016-06-25 13:54:56 +0200
comments: true
categories: arm armv8 qemu embedded
---

Since makers community start to be flooded with ARMv8 development boards it is
highly probable that next years we will start see lot of products based on new
ARM architecture. There are many interesting thing to learn for Embedded
Systems Consultants. There is open source [ARM Trusted Firmware](https://github.com/ARM-software/arm-trusted-firmware) and UEFI
specification already contain support for ARMv8.

Goal of this post is to setup ARMv8 emulated environment for further learning
and development purpose. 

QEMU
----

After code comparison for Linaro and upstream version of QEMU I decide that
upstream contain much more commits related to ARMv8.

```
git clone git://git.qemu.org/qemu.git
```

### Python requirement

If you faced message like this:

```
ERROR: Cannot use 'python', Python 2.6 or later is required.
       Note that Python 3 or later is not yet supported.
       Use --python=/path/to/python to specify a supported Python.
```

you can use `virtualenv` to avoid problem:

```
virtualenv -p /usr/bin/python2.7 ~/local/py2.7-venv
source ~/local/py2.7-venv/bin/activate
```
