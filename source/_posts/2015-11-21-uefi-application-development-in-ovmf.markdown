---
layout: post
title: "UEFI Application development in OVMF"
date: 2015-11-21 11:18:52 +0100
comments: true
categories: uefi ovmf
---

OVMF (_Open Virtual Machine Firmware_) is a project that aim is to enable UEFI
support in various virutal machines. According to
[whitepaper](http://www.linux-kvm.org/downloads/lersek/ovmf-whitepaper-c770f8c.txt)
various projects are interested in supporting OVFM ie. VirtualBox, Xen, BHyVe
and of course QEMU. Why someone may be interesting in OVMF ?

* IMHO the most important reason is that it give ability to develop UEFI
  applications without hardware or with virtual hardware ie. you can run and
  test configuration application, bootloaders, network aplications and other.
* It enables training and learning environment. Not every one can obtain UEFI
  PC and mess with it when there is risk of destroying sth.
* Many other reason like testing/developing UEFI support for OSes, simplify
  debugging, eliminate dependencies on legacy address space and devices. This
  and other interesting reasons can be found in [Laszlo Ersek whitepaper](http://www.linux-kvm.org/downloads/lersek/ovmf-whitepaper-c770f8c.txt)

What I want to show below if procedure for setting up development environment
for UEFI applications and show how to develop and deploy `Hello World`
application in that environment.

## QEMU

Let's use upstream version of QEMU:

```
git clone https://github.com/qemu/qemu.git 
cd qemu
git submodule init
git submodule update
./configure --target-list=x86_64-softmmu
make -j$(nproc)
```

### Prepare application image

Of course size depends on you application requirements. Usually 128MB is way
too much.

```
./qemu-img create -f qcow2 app.img 128M
sudo mkfs.vfat -s 16 -F 32 app.img
```
