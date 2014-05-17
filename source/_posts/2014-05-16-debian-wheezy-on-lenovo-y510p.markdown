---
layout: post
title: "Linue (Debian Wheezy) on Lenovo y510p
date: 2014-05-16 22:36:27 +0200
comments: true
categories: 
---

After long consideration I choose new laptop. I had about $1000 (3000PLN) and
most important things to were:

* i7 CPU - because of performance (of course at least 4700 series)
* SSD - again performance
* 17.3” - working space
* no OS/FreeDos/Linux - I will not pay additional fee to M$ for system that I won’t use
* Full HD resolution
* at least 8GB RAM
* non-glare display

As it usually in life when came to find such a laptop I figure out that it is
impossible to find SSD+17.3” in my price, so I have to resign from both :(. In
final round I had two candidates [Acer Aspire V7-772G](http://www.notebookcheck.net/Review-Acer-Aspire-V3-772G-747A321-Notebook.93916.0.html)
and [Lenovo IdeaPad y510p](http://www.notebookcheck.net/Review-Lenovo-IdeaPad-Y510p-Notebook.97470.0.html).
After reading Newegg reviews about both laptops I choose Lenovo. It is better
brand and got better design (metal lid), other parameters except RAM expansion
possibilities.

## First boot
Today I have it on my desk and trying to install Debian Wheezy. I put Debian
netinst in drive and get wonderful UEFI message :)

[security image]

I figure out that I have to disable one of my favourite feature from M$ -
namely Secure Boot. I was surprised when I realized that there is no hot key to
enter UEFI Setup. Instead of hot key Lenovo decide to put small button (called
Novo Button) on the side of laptop near power socket. Quite interesting idea
taking into consideration that InsydeH20 Setup Utility doesn’t provide much of
functions, so we will enter maybe dozen of times during laptop lifetime. Also I
think it can improve boot time a little bit because UEFI don’t have to poll for
user input during hot key time window.


## Disable secure thing

So to disable this feature-that-name-should-no-be-repeated you have to enter
Setup using Novo Button and switch to Security tab. At first glance you can
find option called `Scure Boot` set to `[Enabled]`. Description of this option
said `Enabl or Disable Secure Boot support`. Don’t be naive this button won’t
what you want. To disable this devil work you have to `Reset to Seupt Mode` by
pushing enter after choosing this option.

[security tab image]

## Installation

Next surprise after booting netinst (you can do this in UEFI mode) is that
Debian 7.5.0 was unable to detect on board Intel LAN card (you can do this in
UEFI mode) is that Debian 7.5.0 was unable to detect on board LAN card. Maybe I
should rephrase - it was able to detect it but it doesn't contain drivers to
use it. So I installed my Debian over wireless Atheros.

## Xorg crash

After all above I booted my favourite distro and it welcomes me with blinking
cursor. Xorg crashed because:
```
(EE) VESA(0): V_BIOS address 0x0 out of range
(EE) Screen(s) found, but none have a usable configuration
```

During long an unequal battle, which was full of google hits. I figured out
that best way to improve awful situation in Debian stable for y510p is to
upgrade to Sid (unstable). I also installed `nvidia-driver` and `nvidia-kernel-dkms`.

Situation improved a lot because I started to see gdm3 login window.

## Unable to login using gdm3

This one proved to be very simple. Symptom manifested itself by continues
restart of gdm3. Quick look into `$HOME/.xsession-errors` and I found:
``
OpenSSL version mismatch. Built against 1000105f, you have 10001080
``
So I tried to apt-get ssh and problem disappeared. But this wasn’t the end of story.

## Gnome-shell

Now I am able to start Gnome but with some obsolete VESA, so I’m getting
information that my hardware is not capable of launching Gnome-shell.``

