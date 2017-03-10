---
author: Piotr Król
layout: post
title: "Nerves project triage on BeagleBone Black"
date: 2017-03-10 22:53:55 +0100
comments: true
categories:  embedded beagleboneblack linux nerves
---

Recently one of my customers brought to my attention [Nerves](http://nerves-project.org).
It aims to simplify use of Elixir (functional language leveraging Erlang VM) in
embedded systems. This system has couple interesting features that are worth of
research and blog post.

First is booting directly to application which is running in BEAM (Erlang VM).
Nerves project replace systemd process with programming language virtual
machine running application code. Concept is very interesting and I wonder if
someone tried to use that with other VMs ie. JVM.

Second Nerves seems to utilize dual image update procedure. In my opinion any
development of modern embedded system should start with update system. Any
design that you can to your system update arsenal will be useful.

Third, Nerves use Buildroot as build system, which will I'm familiar with.
Using popular build systems means simplified support for huge set of platforms
(at point of writing this article Buildroot have 142 config files).


