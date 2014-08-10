---
layout: post
title: "Debugging coreboot for QEMU armv7 (vexpress-a9) emulated mainboard"
date: 2014-08-07 23:08:39 +0200
comments: true
categories: coreboot
---

Recently I came back to look into coreboot mainly because I figured out that
there is some demand for low level skills
([1](http://bit.ly/1sBSybZ),[2](http://bit.ly/1sBSR6F)). I was surprised that
under the wings of Google coreboot team start to support ARM (BTW next great
skill to learn). So I took latest code compile QEMU armv7 mainboard model and
tried to kick it in latest qemu-system-arm. Unfortunately it didn't boot.

Command for running qemu that I found in one commit log:
```
qemu-system-arm -M vexpress-a9 -m 1024M -nographic -kernel build/coreboot.rom
```
It ends with qemu error:
```
qemu: fatal: Trying to execute code outside RAM or ROM at 0x04000000

R00=00000002 R01=00000000 R02=00000000 R03=00000000
R04=00000000 R05=00000000 R06=00000000 R07=00000000
R08=00000000 R09=00000000 R10=00000000 R11=00000000
R12=00000000 R13=0007fed0 R14=6001032f R15=04000000
PSR=600000d3 -ZC- A svc32
(...)
```

At the beginning I thought that it is a mistake so I tried:
```
qemu-system-arm -M vexpress-a9 -m 1024M -nographic -bios build/coreboot.rom
```
What ends with:
```
qemu: fatal: Trying to execute code outside RAM or ROM at 0xfffffffe

R00=00000002 R01=ffffffff R02=ffffffff R03=ffffffff
R04=ffffffff R05=ffffffff R06=ffffffff R07=ffffffff
R08=00000000 R09=ffffffff R10=ffffffff R11=ffffffff
R12=00000000 R13=0007fed0 R14=0000032f R15=fffffffe
PSR=600000f3 -ZC- T svc32
```

Obviously qemu complains on value in R15 (PC - Program Counter), which is the
address of current instruction (like EIP in x86).
