---
layout: post
title: "Running U-boot on emulated IGEPv2 (DM3830)"
date: 2014-01-30 00:50:07 +0100
comments: true
categories: vdb, embedded
published: false
keywords: u-boot, embedded, igepv2, dm3830
description: 
---

<p class=intro>
First, you can ask why I choose IGEPv2 for this article ? And I have to admit that I was strongly influenced by Free Electrons training
materials that I studied to improve my embedded skills.
</p>

## Qemu from Linaro ##

When I take a close look at [IGEPc2]() site I found interesting [IGEP QEMU Emulator]() package. It contain few interesting things inside:
- IGEP QEMu patch
- igep-nano 512MB image
- few qemu binaries (qemu-ga, qemu-io, qemu-nbd, ...)
Unfortunately Qemu upstream doesn't contain [IGEPv2](https://www.isee.biz/products/igep-processor-boards/igepv2-dm3730) 
support 
