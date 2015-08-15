---
layout: post
title: "Reading QEMU SPD EEPROM in UEFI application - pt1. environment setup"
date: 2015-07-04 14:58:18 +0200
comments: true
categories: UEFI,QEMU,OVMF
---

What is SPD and why we may want to read it ?

[SPD](https://en.wikipedia.org/wiki/Serial_presence_detect) (_Serial Presence
Detect_) is [JEDEC](https://en.wikipedia.org/wiki/JEDEC) standard aims to
formalize way of providing description of dual in-line memory modules (DIMM).
SPD is stored in EEPROM both with DIMMs on one circuit board. Obviously
motherboard vendors cannot predict and hardcode all characteristics of DIMM
modules that can be used with their product, so they need mechanism to identify
each module for purpose of memory controller configuration during boot. Of
course this is useful mostly in systems like PC, workstations and servers. This
market is dominated by x86 architecture with small signs of ARM, but because of
M$ and Secure Boot stuff tied to UEFI.

What kind of information SPD contain ? SPD use 128 bytes to store number of
banks per device, [CAS](https://en.wikipedia.org/wiki/CAS_latency) and other
latency information, refresh timing, vendor specific information and a lot
more. All those are required because DRAM needs constant data refreshing to
preserve its content. More about this process can be ready
[here](https://en.wikipedia.org/wiki/Memory_refresh).

This kind of information usually is needed at very early stage of system
booting (PEI phase in UEFI, ROM stage in Coreboot), but it happens that system
prevent access to SMBus.

