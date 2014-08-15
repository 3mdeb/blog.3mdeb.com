---
layout: post
title: "Coreboot for QEMU armv7 (vexpress-a9) emulated mainboard"
date: 2014-08-07 23:08:39 +0200
comments: true
categories: coreboot
---

Recently I came back to look into coreboot. Mainly because low level is fun and I figured out that
there is some demand for this kind of skills
([first odesk job](http://bit.ly/1sBSybZ), [second odesk job](http://bit.ly/1sBSR6F)). I was surprised that
under the wings of Google coreboot team start to support ARM (BTW ARM programming is IMHO next great
skill to learn). So I took latest code compile QEMU armv7 mainboard model and
tried to kick it in latest qemu-system-arm. Unfortunately it didn't boot. Below you can find my TL;DR story.

## QEMU armv7 compilation - very quick steps
```
git clone http://review.coreboot.org/p/coreboot
cd coreboot
git submodule update --init --checkout
make menuconfig
```

Set: `Mainboard -> Mainboard model -> QEMU armv7 (vexpress-a9)`

NOTE: To prevent annoying warning when running gdb, namely:
```
warning: Can not parse XML target description; XML support was disabled at compile time
```
`libexpat1-dev` should be installed.

```
sudo apt-get install libexpat1-dev
cd util/crossgcc
./buildgcc -y -j 8 -p armv7 -G
cd ../..
make
```

`buildgcc` will provide armv7 toolchain with debugger (`-G`) and compilation
will use 8 parallel jobs.


## qemu-armv7 debugging tips and tricks

* Use good gdbinit, so every instruction gdb will automatically provide most
  useful informations. IMHO good choice is `fG!` gdbinit shared on
  [github](https://github.com/gdbinit/Gdbinit). It contain support for ARM and x86.
  To switch to ARM mode inside gdb simple use `arm` command.
  Output looks pretty awesome:

<a class="fancybox" rel="group" href="/assets/images/gdbinit.png"><img src="/assets/images/gdbinit.png" alt="" /></a>

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
address of current instruction (like EIP in x86). 

Stepping through assembler instructions using cross-compiled debugger
(`util/crossgcc/xgcc/bin/armv7-a-eabi-gdb`) points to:

```
0x6001024f:  ldmia.w sp!, {r2, r3, r4, r5, r6, r7, r9, r10, r11, pc}
```

This instruction cause that PC goes to 0x0 and then run instruction from zeroed
memory, which in ARM instructions means:

```
andeq   r0, r0, r0
```

It happens till PC reach 0x4000000 which is out of 'RAM or ROM' for qemu.
Unfortunately there is no sign about `ldmia` instruction with above range of
registers in coreboot and qemu code.

## Bisection

I knew that at some point qemu worked with coreboot. I tried few versions and
it points me to some commit between v2.1.0-rc1 and v2.1.0-rc0. For `-kernel`
switch I was able to narrow down problem to one commit that change
`VE_NORFLASHALIAS` option for vexpress-a9 to 0
([6ec1588](http://git.qemu.org/?p=qemu.git;a=commit;h=6ec1588e09770ac7e9c60194faff6101111fc7f0)).
It looks like for vexpress-a9 qemu place kernel at 0x60000000
(vexpress.highmem), which is aliased to range 0x0-0x3ffffff.
`VE_NORFLASHALIAS=0` cause mapping of vexpress.flash0 to the same region as
kernel and because flash (`-bios`) was not added we have empty space (all
zeros) what gives `andeq r0, r0, r0`.

Right now I have working version of coreboot but only with `-kernel` and
`VE_NORFLASHALIAS=-1` set in hw/arm/vexpress.c. The main questions are:

* what is the correct memory map for qemu-armv7 and how coreboot should be mapped ?
* what's going on with coreboot or qemu that I can't go through bootblock ?

## Debugging

Coreboot as UEFI has few phases. For UEFI we distinguish SEC, PEI, DXE and BDS
(there are also TSL, RT and AL, but not important for this considerations). On
coreboot side we have bootblock, romstage, ramstage and payload.

### qemu-armv7 bootblock failure

qemu-armv7 booting procedure start from `_rom` section which contain hardcoded
jump to `reset` procedure. After that go through few methods like on below flow:

```
_rom
|-> reset
    |-> init_stack_loop
        |-> call_bootblock
            |-> main
                |-> armv7_invalidate_caches
                    |-> icache_invalidate_all
                    |-> dcache_invalidate_all
                      |-> dcache_foreach
```

At the end of `dcache_foreach` we experience failure because `ldmia`
instruction tries to restore registers from stack, which should be stored at
the beginning of `dcache_foreach`, by:

```
stmdb›  sp!, {r0, r1, r4, r5, r6, r7, r9, sl, fp, lr}
```

Unfortunately for some reason stack doesn't contain any reasonable values (all
0xffffffff). Why is that ?

