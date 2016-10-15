---
layout: post
title: "Debugging OptionROM in QEMU"
date: 2016-10-15 01:24:12 +0200
comments: true
categories: BIOS OpROM x86 QEMU coreboot
---

Below I describe my experience with debugging custom ROM using using QEMU and
coreboot. There are many reason why this maybe needed.

This post was inspired by work made by [Darmawan Salihun](http://bioshacking.blogspot.com/2011/10/pci-option-rom-debugging-with-seabios.html).

I used QEMU from Debian sid repository and coreboot compiled from scratch for QEMU (``).

To add ROM binary to image:

```
./build/cbfstool build/coreboot.rom add -f test.rom -n genroms/test.rom -t raw
```

To run QEMU with coreboot binary:

```

```

