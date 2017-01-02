---
author: Piotr Król
layout: post
title: "How to add debugging logs to libmosquitto"
date: 2016-12-31 00:00:38 +0100
comments: true
categories: mosquitto mqtt yocto linux
---

Below I described how to enable debugging messages to `libmosquitto` as this is
not trivial based on documentation found in Google. The need for debugging
message born when we faced AWS IoT issues on our Embedded Linux platform. So
below I will show how to build `libmosquitto` in Poky distro, but this
knowledge can be applied to typical build.
