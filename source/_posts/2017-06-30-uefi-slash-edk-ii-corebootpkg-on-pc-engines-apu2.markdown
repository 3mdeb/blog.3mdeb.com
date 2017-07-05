---
author: Piotr Król
layout: post
title: "UEFI/EDK II CorebootPkg on PC Engines APU2"
date: 2017-06-30 22:14:55 +0200
comments: true
categories: coreboot uefi edk2 apu2
---

Recently we were reached by person interested in running CoreOS on APU2. CoreOS
is very interesting system from security point of view it was created to
support containers out of the box.

But what is more interesting to firmware developer is idea of booting
UEFI-aware operating system no matter if it is CoreOS or Debian. Because of
that we decided to triage [coreboot UEFI payload](https://github.com/tianocore/tianocore.github.io/wiki/Coreboot_UEFI_payload)
on PC Engines APU2 platform.

For interested in that topi I recommend to take look at [video from coreboot conference 2016](https://youtu.be/I08NHJLu6Us?list=PLiWdJ1SEk1_AfMNC6nD_BvUVCIsHq6f0u).

All my modification from below article can be found in [3mdeb edk2 fork](https://github.com/3mdeb/edk2/tree/apu2-uefi)

## Firmware

Let's start with building APU2 mainline. First follow [this instruction](https://github.com/pcengines/release_manifests)
and build mainline version of coreboot. Meanwhile you can take care of EDK2 CorebootPkg build:

```
git clone https://github.com/tianocore/edk2.git
cd edk2
source edksetup.sh
```

On my Debian testing I had to explicitly change compilers to `gcc-5` and `g++-5`:

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 10
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 10
```

Otherwise build fails. After that I was able to build `UEFIPAYLOAD.fd`:

```
make -c BaseTools
build -a IA32 -p CorebootPayloadPkg/CorebootPayloadPkgIa32.dsc -b DEBUG -t GCC5
```

Build result is located in
`Build/CorebootPayloadPkgIA32/DEBUG_GCC5/FV/UEFIPAYLOAD.fd`. Following [build and integration instructions](https://raw.githubusercontent.com/tianocore/edk2/master/CorebootPayloadPkg/BuildAndIntegrationInstructions.txt)
I added build result as `An ELF executable payload`.

{image tbd}

It is important to deselect secondary payloads like `memtest86+` and
`sortbootorder` to avoid compilation issues.

### EDK2 development

For all interested in EDK2 code development I strongly advise to follow [Laszlo's  guide](https://github.com/tianocore/tianocore.github.io/wiki/Laszlo's-unkempt-git-guide-for-edk2-contributors-and-maintainers).

Useful script in case of APU2 may be:

```sh
#!/bin/bash
build -a IA32 -p CorebootPayloadPkg/CorebootPayloadPkgIa32.dsc -b DEBUG -t GCC5
cp Build/CorebootPayloadPkgIA32/DEBUG_GCC5/FV/UEFIPAYLOAD.fd ../apu2_fw_rel/apu2/coreboot/
cd ../apu2_fw_rel
./apu2/apu2-documentation/scripts/apu2_fw_rel.sh build-ml
read -n 1 -s -p "Press any key to continue with flashing recent build ..."
./apu2/apu2-documentation/scripts/apu2_fw_rel.sh flash-ml root@192.168.0.105
cd ../edk2
```

I added prompt for flashing since it happen to forget remove recovery flash,
what may lead to blocking further development since both SPIs will contain not
bootable firmware.


## Booting

### CbSupportDxe assert

First problem I faced was assert in `CbSupportDxe.c` related to adding 1MB
memory for LAPIC. Reservation happen in CbSupportDxe entry point.

```
Loading driver at 0x000CFC25000 EntryPoint=0x000CFC2595B CbSupportDxe.efi
InstallProtocolInterface: BC62157E-3E33-4FEC-9920-2D3B36D750DF CFC42190
ProtectUefiImageCommon - 0xCFC42928
  - 0x00000000CFC25000 - 0x0000000000002580
PROGRESS CODE: V03040002 I0
Failed to add memory space :0xFEE00000 0x100000

ASSERT_EFI_ERROR (Status = Access Denied)

DXE_ASSERT!: [CbSupportDxe] /home/pietrushnic/storage/wdc/projects/2017/pcengines/apu/src/edk2/CorebootModulePkg/CbSupportDxe/CbSupportDxe.c (57): !EFI_ERROR (Status)
```

Interestingly CbSupportDxe seems to not have problem with finding GUID HOB,
which is right after asserting reservation. It also pass installation of ACPI
and SMBIOS table. I just commented that reservation and moved forward to see
what will happen.

### PcRtcEntry assert

Next problem happen during initialization of RTC:

```
Loading driver at 0x000CFE50000 EntryPoint=0x000CFE507DB PcRtc.efi
InstallProtocolInterface: BC62157E-3E33-4FEC-9920-2D3B36D750DF CFC2AE90
ProtectUefiImageCommon - 0xCFC2AB28
  - 0x00000000CFE50000 - 0x0000000000003980
  PROGRESS CODE: V03040002 I0
  ASSERT_EFI_ERROR (Status = Device Error)

  DXE_ASSERT!: [PcRtc] /home/pietrushnic/storage/wdc/projects/2017/pcengines/apu/src/edk2/PcAtChipsetPkg/PcatRealTimeClockRuntimeDxe/PcRtcEntry.c (143): !EFI_ERROR (Status)
```

Unfortunately Real Time Clock is required architecture protocol and cannot be
omitted.

First problem with this code was incorrect state of `Valid RAM and Time` bit in
RTC Date Alarm register (aka Register D).
