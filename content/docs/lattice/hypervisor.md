+++
title = "A Cloud-Native, Distributed WebAssembly Hypervisor"
linktitle = "Cloud Native Hypervisor"
date = 2020-06-10

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 97

[menu.docs]
  parent = "lattice"
  weight = 3
+++

> "A hypervisor (or virtual machine monitor, VMM) is computer software, firmware or hardware that creates and runs virtual machines." - [Wikipedia](https://en.wikipedia.org/wiki/Hypervisor).

As discussed in the previous section, when building our code, we want to be able to write our business logic in a way that is _no longer tightly coupled to a host OS process_. Moreover, we want our deployment artifacts to be environment-agnostic, and not just in a way that is hidden behind automation scripts and configuration. We truly want the code we write to be able to flow freely from our laptops to the cloud to the edge and even to IoT and embedded devices.

[waSCC](https://github.com/wascc), through a combination of [WebAssembly](https://webassembly.org) and other technologies, gives us a _Cloud Native Hypervisor_ with its **lattice** feature. What a hardware hypervisor provides for virtual machines, waSCC's lattice provides for [actors](../../concepts/actors).
