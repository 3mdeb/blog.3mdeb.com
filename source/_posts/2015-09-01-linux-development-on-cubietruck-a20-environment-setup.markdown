---
layout: post
title: "Linux development on Cubietruck (A20) - environment setup"
date: 2015-09-01 21:48:55 +0200
comments: true
categories: linux embedded cubietruck A20
---

During last couple of months I see pretty big interest in building products on
[A20](http://linux-sunxi.org/A20) SoC. This is pretty decent chip which can be
bought for 6USD in quantity. Most important features are:

* Dual-Core ARM Cortex-A7 (ARMv7)
* Mali-400 MP2
* HDMI, VGA and LCD
* MMC and NAND
* OTG and 2 Host ports

Tracking media related to low-end mobile market IMHO the hottest SoCs are
Allwinner A20 and Rockchip RK3288.

A20 ship with many development boards like Cubieboard or pcDuino series, Banana
Pi, MarsBoard or Hummingbird. About a year ago I choose to buy Cubietruck and
this lead to couple interesting projects from porting
[USBSniffer](http://elinux.org/BeagleBoard/GSoC/2010_Projects/USBSniffer) to
writing bare-metal bootloader based on U-boot code. Because I figure out that
it is hard to me to recover Cubietruck setup that I made for kernel development
I decided to write this post and leave notes for me and others who want quickly
bootstrap Cubietruck setup.

TODO: picture

##Prerequisites

* Linux development workstation (I use Debian stretch/sid)
* USB to TTL serial adapter - best would be with original FT232RL, but Chinese
  substitutes also can work
* microSD card
* Ethernet cable - to connect your CT to router
* good power supply 5V/2.5A - USB should also work when taking care about power
  budget of whole setup
* HDMI or VGA monitor - nice to have

##General approach

Bootloader (U-Boot) obtain IP address dynamically then using hard coded
information about TFTP server it downloads script which contain instruction
about next steps. Usually next steps include downloading kernel over TFTP and
passing to it parameter indicating that rootfs should be mounted over NFS.

Below I put together various pieces spread across network to have it in one
place.

##Quick TFTP setup

```
sudo apt-get install tftpd-hpa
```

To check if TFTP listen:

```
[23:12:39] pietrushnic:~ $ netstat -an|grep :69
udp        0      0 0.0.0.0:69              0.0.0.0:* 
```

It would be useful to have separate directory if in future setup will be enhanced for other boards:

```
sudo mkdir /srv/tftp/ct
```

##Quick NFS setup

```
sudo apt-get install nfs-kernel-server
sudo mkdir /srv/nfs
sudo vim /etc/exports
```

Add line like this:
```
/srv/nfs       *(rw,sync,no_root_squash,no_subtree_check)
```

```
sudo service nfs-kernel-server restart
```

<a name="toolchain"></a>
## Toolchain

I'm using Linaro toolchain based on GCC4.9 you can download package
[here](http://releases.linaro.org/15.05/components/toolchain/binaries/arm-linux-gnueabihf/).

```
tar xf gcc-linaro-4.9-2015.05-x86_64_arm-linux-gnueabihf.tar.xz
export PATH=${PATH}:${PWD}/gcc-linaro-4.9-2015.05-x86_64_arm-linux-gnueabihf/bin/
```

To verify this step you can try:

```
[23:24:01] pietrushnic:~ $ arm-linux-gnueabihf-gcc   
arm-linux-gnueabihf-gcc: fatal error: no input files
compilation terminated.
```

##U-Boot

```
git clone git://git.denx.de/u-boot.git
cd u-boot
make CROSS_COMPILE=arm-linux-gnueabihf- Cubietruck_defconfig
make CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
```

Log like this:

```
[23:42:38] pietrushnic:u-boot git:(master) $ make CROSS_COMPILE=arm-linux-gnueabihf- -j8
make: arm-linux-gnueabihf-gcc: Command not found
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
dirname: missing operand
Try 'dirname --help' for more information.
scripts/kconfig/conf  --silentoldconfig Kconfig
  CHK     include/config.h
  UPD     include/config.h
  GEN     include/autoconf.mk
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
  GEN     include/autoconf.mk.dep
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
scripts/Makefile.autoconf:47: recipe for target 'include/autoconf.mk.dep' failed
make[1]: *** [include/autoconf.mk.dep] Error 1
make[1]: *** Waiting for unfinished jobs....
scripts/Makefile.autoconf:72: recipe for target 'include/autoconf.mk' failed
make[1]: *** [include/autoconf.mk] Error 1
  GEN     spl/include/autoconf.mk
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
scripts/Makefile.autoconf:75: recipe for target 'spl/include/autoconf.mk' failed
make[1]: *** [spl/include/autoconf.mk] Error 1
make: *** No rule to make target 'include/config/auto.conf', needed by 'include/config/uboot.release'.  Stop.
```
means that you incorrectly set [toolchain](#toolchain).

##Linux kernel

###sunxi-3.4 kernel

```
git clone -b sunxi-3.4 --depth 1 https://github.com/linux-sunxi/linux-sunxi.git
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- sun7i_defconfig
make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage modules
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=output modules_install
```

###Mainline kernel (sunxi-next 387a2c191af6 4.2.0-rc4)

```
git clone git://github.com/mripard/linux.git -b sunxi-next
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- sunxi_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage dtbs
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=output modules modules_install
```

<a name="debian_rootfs"></a>
##Rootfs

Based on [Olimex guide](https://olimex.wordpress.com/2014/07/21/how-to-create-bare-minimum-debian-wheezy-rootfs-from-scratch/).

```
sudo apt-get install qemu-user-static
targetdir=rootfs
distro=testing
mkdir $targetdir
sudo debootstrap --arch=armhf --foreign $distro $targetdir
sudo cp /usr/bin/qemu-arm-static $targetdir/usr/bin/
sudo cp /etc/resolv.conf $targetdir/etc
sudo chroot $targetdir /bin/bash -i
distro=testing
export LANG=C
cat <<EOT > /etc/apt/sources.list
deb http://httpredir.debian.org/debian $distro main contrib non-free
deb-src http://httpredir.debian.org/debian $distro main contrib non-free
EOT
apt-get update
apt-get install locales dialog
dpkg-reconfigure locales
```

In this place I prefer to use en_US.UTF-8 as a _lingua franca_ of Linux world.
Next we will install couple of tools that are almost always useful.

```
apt-get install openssh-server ntpdate git vim
passwd
echo <<EOT >> /etc/network/interfaces
auto eth0
allow-hotplug eth0
iface eth0 inet dhcp
EOT
echo cubietruck > /etc/hostname
exit
sudo rm $targetdir/etc/resolv.conf
sudo rm $targetdir/usr/bin/qemu-arm-static
```

At this point it is good to make a backup copy. Time consuming but with very
small output:

```
sudo XZ_OPT=-9 tar cJf rootfs.tar.xz rootfs
```

##Let's put it all together

###Prepare SD card

Assuming your SD card is on `/dev/sdc`

```
sudo umount /dev/sdc1 #umount any automatically mounted partitions
card=/dev/sdc
sudo dd if=/dev/zero of=${card} bs=1M count=1 #clean partition table
```

Flash U-Boot image:

```
sudo dd if=u-boot-sunxi-with-spl.bin of=${card} bs=1024 seek=8 
```

Partitioning:

```
cat << EOT | sudo sfdisk ${card}
1,16,c
,,L
EOT
```

## Toolchain

I'm using Linaro toolchain based on GCC4.9 you can download package
[here](http://releases.linaro.org/15.05/components/toolchain/binaries/arm-linux-gnueabihf/).

```
tar xf gcc-linaro-4.9-2015.05-x86_64_arm-linux-gnueabihf.tar.xz
export PATH=${PATH}:${PWD}/gcc-linaro-4.9-2015.05-x86_64_arm-linux-gnueabihf/bin/
```

To verify this step you can try:

```
[23:24:01] pietrushnic:~ $ arm-linux-gnueabihf-gcc   
arm-linux-gnueabihf-gcc: fatal error: no input files
compilation terminated.
```

##U-Boot

```
git clone git://git.denx.de/u-boot.git
cd u-boot
make CROSS_COMPILE=arm-linux-gnueabihf- Cubietruck_defconfig
make CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
```

Log like this:

```
[23:42:38] pietrushnic:u-boot git:(master) $ make CROSS_COMPILE=arm-linux-gnueabihf- -j8
make: arm-linux-gnueabihf-gcc: Command not found
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
dirname: missing operand
Try 'dirname --help' for more information.
scripts/kconfig/conf  --silentoldconfig Kconfig
  CHK     include/config.h
  UPD     include/config.h
  GEN     include/autoconf.mk
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
  GEN     include/autoconf.mk.dep
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
scripts/Makefile.autoconf:47: recipe for target 'include/autoconf.mk.dep' failed
make[1]: *** [include/autoconf.mk.dep] Error 1
make[1]: *** Waiting for unfinished jobs....
scripts/Makefile.autoconf:72: recipe for target 'include/autoconf.mk' failed
make[1]: *** [include/autoconf.mk] Error 1
  GEN     spl/include/autoconf.mk
/bin/sh: 1: arm-linux-gnueabihf-gcc: not found
scripts/Makefile.autoconf:75: recipe for target 'spl/include/autoconf.mk' failed
make[1]: *** [spl/include/autoconf.mk] Error 1
make: *** No rule to make target 'include/config/auto.conf', needed by 'include/config/uboot.release'.  Stop.
```
means that you incorrectly set [toolchain](#toolchain).

##Linux kernel

###sunxi-3.4 kernel

```
git clone -b sunxi-3.4 --depth 1 https://github.com/linux-sunxi/linux-sunxi.git
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- sun7i_defconfig
make -j$(nproc) ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage modules
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=output modules_install
```

###Mainline kernel (sunxi-next 387a2c191af6 4.2.0-rc4)

```
git clone git://github.com/mripard/linux.git -b sunxi-next
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- sunxi_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage dtbs
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=output modules modules_install
```

<a name="debian_rootfs"></a>
##Rootfs

Based on [Olimex guide](https://olimex.wordpress.com/2014/07/21/how-to-create-bare-minimum-debian-wheezy-rootfs-from-scratch/).

```
sudo apt-get install qemu-user-static
targetdir=rootfs
distro=testing
mkdir $targetdir
sudo debootstrap --arch=armhf --foreign $distro $targetdir
sudo cp /usr/bin/qemu-arm-static $targetdir/usr/bin/
sudo cp /etc/resolv.conf $targetdir/etc
sudo chroot $targetdir /bin/bash -i
distro=testing
export LANG=C
cat <<EOT > /etc/apt/sources.list
deb http://httpredir.debian.org/debian $distro main contrib non-free
deb-src http://httpredir.debian.org/debian $distro main contrib non-free
EOT
apt-get update
apt-get install locales dialog
dpkg-reconfigure locales
```

In this place I prefer to use en_US.UTF-8 as a _lingua franca_ of Linux world.
Next we will install couple of tools that are almost always useful.

```
apt-get install openssh-server ntpdate git vim
passwd
echo <<EOT >> /etc/network/interfaces
auto eth0
allow-hotplug eth0
iface eth0 inet dhcp
EOT
echo cubietruck > /etc/hostname
exit
sudo rm $targetdir/etc/resolv.conf
sudo rm $targetdir/usr/bin/qemu-arm-static
```

At this point it is good to make a backup copy. Time consuming but with very
small output:

```
sudo XZ_OPT=-9 tar cJf rootfs.tar.xz rootfs
```

##Let's put it all together

###Prepare SD card

Assuming your SD card is on `/dev/sdc`

```
sudo umount /dev/sdc1 #umount any automatically mounted partitions
card=/dev/sdc
sudo dd if=/dev/zero of=${card} bs=1M count=1 #clean partition table
```

Flash U-Boot image:

```
sudo dd if=u-boot-sunxi-with-spl.bin of=${card} bs=1024 seek=8 
```

Partitioning:

```
cat << EOT | sudo sfdisk ${card}
1,16,c
,,L
EOT
```

<a name="known_issues"></a>
## Known issues

###TFTP error: 'Unsupported option(s) requested' (8)

This problem was discussed [here](http://lists.denx.de/pipermail/u-boot/2015-August/225129.html).
You can fix it by changing TFTP `TIMEOUT`:

```
diff --git a/net/tftp.c b/net/tftp.c
index 18ce84c20214..33fe4e47a616 100644
--- a/net/tftp.c
+++ b/net/tftp.c
@@ -19,7 +19,7 @@
 /* Well known TFTP port # */
 #define WELL_KNOWN_PORT        69
 /* Millisecs to timeout for lost pkt */
-#define TIMEOUT                100UL
+#define TIMEOUT                1000UL
 #ifndef        CONFIG_NET_RETRY_COUNT
 /* # of timeouts before giving up */
 # define TIMEOUT_COUNT 1000
```
