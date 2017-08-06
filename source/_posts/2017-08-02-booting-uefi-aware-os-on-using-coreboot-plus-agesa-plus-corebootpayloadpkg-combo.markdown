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

On my workbench I had available mSATA disks and USB stick. Knowing that USB
stick contain Voyage Linux, that already worked with apu2 and coreboot, I
decided go USB path.

## Enabling USB on apu2
