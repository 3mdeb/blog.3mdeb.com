---
layout: post
title: "Linux RPi and USB over IP"
date: 2014-08-18 21:26:37 +0200
comments: true
categories: raspberry-pi, linux, usb
---

Recently I came to the idea of remote access for embedded development kits like
[Arduino](http://www.arduino.cc/), [RPi](http://www.raspberrypi.org/) or more
powerful [Jetson TK1](https://developer.nvidia.com/jetson-tk1). During
exploration of remote access for embedded boards I figured out that as usually
someone else thought about this before:

* [REMBL - Remote Embedded Lab](https://www.youtube.com/watch?v=F6ZUVKnr-Qk)
* [REMBL 1st phase offer](https://www.youtube.com/watch?v=0SHeJgbC0FQ)
* [Google Groups post](https://groups.google.com/forum/#!topic/omapdiscuss/YF9024BwR_0)

I wrote to Ayman (the author of REMBL idea) to ask about the project and
because I found him to be open source and collaborative development enthusiast I decide
to write this post. So our short e-mail conversation leads us to USB over IP topic.
Why ? Simply because it is most used interface that almost all boards have included.

## USB over IP

Trying to google this term doesn't give much except many companies that give
you it as a service. This give some information about some potential on the
market. Main idea is well presented on open source project page for [usbip](http://usbip.sourceforge.net/).
