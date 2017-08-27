---
author: Piotr Król
layout: post
title: "Booting UEFI-aware OS using coreboot+AGESA+CorebootPayloadPkg combo"
date: 2017-08-02 14:15:02 +0200
comments: true
categories: agesa coreboot uefi edk2 debian
---

After publishing [blog post about enabling UEFI Shell on apu2]() I decided to
take next step and try to install UEFI-aware operating system like recent
Debian 9.

Enabling UEFI Shell left firmware in state in which most commands work
correctly, but no device except serial was detected. Obviously it would be
great to have some storage or at least diskless installation that we can boot
on apu2. Speaking about diskless network interfaces were also not visible.

Because all important devices are connected on PCIe bus I decided to figure out
why I can't see anything. Quick check revealed that root bright is not
detected.

## Enabling PCI on apu2

I started with investigation of memory ranges disabled during work on UEFI
Shell booting. Call to reserve LAPIC memory range hardcoded to `0xfee00000`
caused assert what was the reason for disabling this code initially.
