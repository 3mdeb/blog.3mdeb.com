---
layout: post
title: "0x1: Qemu as an environment for embedded board emulation"
date: 2013-06-07 08:27
comments: true
categories: [embedded, linux, vdb, qemu]
published: true
keywords: virtual development board, Qemu, Linux, Emulation
description: How to compile Qemu and brief embedded Linux history
---
## Table of contents ##

* [Introduction](/blog/2013/06/07/qemu-as-an-environment-for-embedded-board-emulation/#intro)
* [Compilation](/blog/2013/06/07/qemu-as-an-environment-for-embedded-board-emulation/#compilation)
* [Kudos](/blog/2013/06/07/qemu-as-an-environment-for-embedded-board-emulation/#kudos)

<a id="intro"></a>
### Introduction ###

QEMU is a CPU emulator using dynamic binary translation to convert guest CPU 
instructions into host CPU instructions[[1]](http://infoscience.epfl.ch/record/149975/files/x86-llvm-translator-chipounov_2.pdf). It supports many architectures from 
x86, through ARM and MIPS, to MicroBlaze. According to compilation configuration 
target list QEMU targets 26 different softmmu types.  Only for ARM it supports 
33 machines (like ARM Versatile/PB (ARM926EJ-S) or Samsung NURI board 
(Exynos4210)) and 28 CPUs (with cortex-a9 and pxa270). It gives access to 
network, storage, video, usb, serial and other peripheral, also user defined. It
is developed under GNU GPL, so everybody are free to make modifications, improve
and extend it. This properties makes QEMU very good candidate for virtual board 
emulator.

<a id="compilation"></a>
### Compilation ###
Let's start creating our Virtual Development Board. As usually I will use latest
greatest version from git:
```bash
git clone http://git.qemu.org/git/qemu.git
```
Compile it and install. Right now I will use only `arm-softmmu` target because it 
will emulate whole arm system for me. ARM right now dominated big part of 
embedded market but we will see if situation won't change in feature.
```bash
cd qemu
./configure --target-list=arm-softmmu
make
make install
```
During configuration process you can encounter lack of `pixman`, just accept qemu 
offer to initialize it as a submodule.
```
git submodule update --init pixman
make # restart compilation process
make install
```
If compilation ends without problem than our first component is ready to use.
Right now we can emulate our ARM based board with many types of CPUs. List of 
all available can be retrieved by running command `qemu-system-arm -cpu ?`, list 
of emulated machines by `-M ?`. Now, let's talk about [toolchains](/blog/2013/06/07/toolchain-for-virtual-development-board).

<a id="kudos"></a>
### Kudos ###
[1] [Dynamically Translating x86 to LLVM using QEMU](http://infoscience.epfl.ch/record/149975/files/x86-llvm-translator-chipounov_2.pdf) 
