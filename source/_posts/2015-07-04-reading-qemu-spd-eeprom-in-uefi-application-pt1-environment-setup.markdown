---
layout: post
title: "Reading QEMU SPD EEPROM in UEFI application - pt1. environment setup"
date: 2015-07-04 14:58:18 +0200
comments: true
categories: UEFI,QEMU,OVMF
---

Let's start with: what is SPD and why we want to read it ?
[SPD](https://en.wikipedia.org/wiki/Serial_presence_detect) (_Serial Presence
Detect_) is [JEDEC](https://en.wikipedia.org/wiki/JEDEC) standard aims to
formalize way of dual in-line memory modules (DIMM) description. SPD is 
stored in EEPROM both with DIMMs on one circuit board. Obviously motherboard
vendors cannot predict and hardcode all characteristics of DIMM modules that
can be used with their product, so they need mechanism to identify each module
for purpose of memory controller configuration during boot. This configuration
is handled by UEFI firmware.

What kind of information SPD contain ? SPD in 128 bytes store number of banks
per device, [CAS](https://en.wikipedia.org/wiki/CAS_latency) and other latency
information, refresh timing, vendor specific information and a lot more.

