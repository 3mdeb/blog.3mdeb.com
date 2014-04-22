---
layout: post
title: "Arduino C tutorial - part 1"
date: 2014-04-19 00:15:57 +0200
comments: true
categories: arduino, c
published: false
---

This tutorial is addressed to Linux users that want to learn how to program
Arduino using plain C. I assume that you own Arduino board and have some basic
understanding of how it works. I prepared my examples with Anduino Duemilanove.
I'm using Debian unstable but all Debian-based distros should work pretty fine.

My inspiration for this series was article by Javier Valcarce [Program Arduino
with AVR-GCC](http://www.javiervalcarce.eu/wiki/Program_Arduino_with_AVR-GCC),
because it was written over 5 years ago I would like to refresh some content
presented there.

_Disclaimer_: I'm not considering myself expert in programming Arduino. I tried
my best to provide high quality materials but nobody's perfect, so if you found
any confusing informations please let me know.

###Beginner notes###
* avr-libc came with its own man application, to find anything in avr-libc
  manual use `arv-man` i.e. `avr-man _BV`
###Hello world or blinking diode###
After setting up Arduino for programming using official IDE you are ready to go
with some basic C code.


\_BV macro means "convert bit number into byte value", it return 1 << number.
