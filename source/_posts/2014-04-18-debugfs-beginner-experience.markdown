---
layout: post
title: "Debugfs - beginner experience"
date: 2014-04-18 21:51:37 +0200
comments: true
categories: linux
published: false
---

In short _debugfs_ is a interface in Linux kernel used by developers to expose
debugging informations to user space.

##Basic usage##
To use _debugfs_ its support should be compiled into kernel. To verify if we
have such a support run:

```
grep CONFIG_DEBUG_FS /boot/config-`uname -r``
```

`CONFIG_DEBUG_FS=y` means that support was compiled in. Default directory to
mount _debugfs_ is `/sys/kernel/debug` but you can use any other by providing
your mount point to command:

```
mount -t debugfs none /media/debug
```
