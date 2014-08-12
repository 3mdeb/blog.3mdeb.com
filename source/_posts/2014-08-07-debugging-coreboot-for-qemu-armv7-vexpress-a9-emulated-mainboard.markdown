---
layout: post
title: "Coreboot for QEMU armv7 (vexpress-a9) emulated mainboard"
date: 2014-08-07 23:08:39 +0200
comments: true
categories: coreboot
---

Recently I came back to look into coreboot mainly because I figured out that
there is some demand for low level skills
([first odesk job](http://bit.ly/1sBSybZ), [second odesk job](http://bit.ly/1sBSR6F)). I was surprised that
under the wings of Google coreboot team start to support ARM (BTW ARM programming is IMHO next great
skill to learn). So I took latest code compile QEMU armv7 mainboard model and
tried to kick it in latest qemu-system-arm. Unfortunately it didn't boot. 

## QEMU armv7 compilation - very quick steps
```
git clone http://review.coreboot.org/p/coreboot
cd coreboot
git submodule update --init --checkout
make menuconfig
```

Set: `Mainboard -> Mainboard model -> QEMU armv7 (vexpress-a9)`

```
cd util/crossgcc
./buildgcc -y -j 8 -p armv7
cd ../..
make
```

## Noob dead end

Command for running qemu that I found in early qemu-armv7 commit log:
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
address of current instruction (like EIP in x86). After digging a while I
figured out that I am unable to narrow down the problem. Mainly because I
cannot find issuer of instruction that cause problem. Namely:
```
0x6001024f:  ldmia.w sp!, {r2, r3, r4, r5, r6, r7, r9, r10, r11, pc}
```

This instruction cause that PC goes to 0 and then run instruction from zeroed
memory, which in ARM instructions means:

```
andeq   r0, r0, r0
```

It happens till PC reach 0x4000000 which is out of 'RAM or ROM' for qemu.
Unfortunately there is no sign about `ldmia` instruction with above range of
registers in coreboot and qemu code. This is probably result of compiler
optimization.

## Bisection

For `-kernel` switch I was able to narrow down problem to one commit that
change `VE_NORFLASHALIAS` option for vexpress-a9 to 0
([6ec1588](http://git.qemu.org/?p=qemu.git;a=commit;h=6ec1588e09770ac7e9c60194faff6101111fc7f0)).
It looks like for vexpress-a9 qemu place kernel at 0x60000000
(vexpress.highmem), which is aliased to range 0x0-0x3ffffff. `VE_NORFLASHALIAS=0`
cause mapping of vexpress.flash0 to the same region as kernel and because flash
(`-bios`) was not added we have empty space (all zeros) what gives `andeq r0, r0, r0`.

