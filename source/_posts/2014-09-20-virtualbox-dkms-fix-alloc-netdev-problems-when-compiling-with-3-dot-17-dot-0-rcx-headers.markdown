---
layout: post
title: "virtualbox-dkms: fix alloc_netdev problems when compiling with 3.17.0-rcX headers"
date: 2014-09-20 22:55:00 +0200
comments: true
categories: virtualbox, debian, linux
---

Intro
=====

Because of my bug hunting approach of using latest kernel I experienced problem
with compiling VirtualBox modules with `3.17.0-rc5` version on my Debian Jessie. Issue is well
known and described for examples [here](https://bugs.launchpad.net/ubuntu/+source/virtualbox/+bug/1358157).
Problem manifest itself with:

```
------------------------------
Deleting module version: 4.3.14
completely from the DKMS tree.
------------------------------
Done.
Loading new virtualbox-4.3.14 DKMS files...
Building only for 3.17.0-rc5+
Building initial module for 3.17.0-rc5+
Error! Bad return status for module build on kernel: 3.17.0-rc5+ (x86_64)
Consult /var/lib/dkms/virtualbox/4.3.14/build/make.log for more information.
Job for virtualbox.service failed. See 'systemctl status virtualbox.service' and 'journalctl -xn' for details.
invoke-rc.d: initscript virtualbox, action "restart" failed.
```

during `virtualbox-dkms` package installation or reconfiguration. In `make.log` you will find compilation error:

```
  CC [M]  /var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.o
/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.c: In function ‘vboxNetAdpOsCreate’:
/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.c:186:48: error: macro "alloc_netdev" requires 4 arguments, but only 3 given
                            vboxNetAdpNetDevInit);
                                                ^
/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.c:184:15: error: ‘alloc_netdev’ undeclared (first use in this function)
     pNetDev = alloc_netdev(sizeof(VBOXNETADPPRIV),
               ^
/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.c:184:15: note: each undeclared identifier is reported only once for each function it appears in
/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.c: At top level:
/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.c:159:13: warning: ‘vboxNetAdpNetDevInit’ defined but not used [-Wunused-function]
 static void vboxNetAdpNetDevInit(struct net_device *pNetDev)
             ^
scripts/Makefile.build:257: recipe for target '/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.o' failed
make[2]: *** [/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp/linux/VBoxNetAdp-linux.o] Error 1
scripts/Makefile.build:404: recipe for target '/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp' failed
make[1]: *** [/var/lib/dkms/virtualbox/4.3.14/build/vboxnetadp] Error 2
Makefile:1373: recipe for target '_module_/var/lib/dkms/virtualbox/4.3.14/build' failed
make: *** [_module_/var/lib/dkms/virtualbox/4.3.14/build] Error 2
make: Leaving directory '/usr/src/linux-headers-3.17.0-rc5+'
```

For sure we have to wait for some time before new version of kernel and
VirtualBox will catch up each other in Debian.

Masochist way to solve issue
----------------------------

Let's get get virtualbox package source, fix issues rebuild package and install
in the system. Patch to apply can be found [here](https://forums.virtualbox.org/viewtopic.php?p=296650#p296650).
According to [PTS](https://packages.qa.debian.org/v/virtualbox.html) `virtualbox-dkms` and other packages can be built from repository that we clone with:

```
git clone git://anonscm.debian.org/pkg-virtualbox/virtualbox.git
```

