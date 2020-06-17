+++
title = "Lattice Overview"
linktitle = "Lattice Overview"
date = 2020-06-10

toc = false  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 95

[menu.docs]
  parent = "lattice"
  weight = 1
+++

A _lattice_ is a distributed, self-forming, self-healing cluster of **waSCC** host runtimes. When running in a lattice, the waSCC host runtime will perform all communication between actors and capability providers over a secure message bus. This one simple change in how waSCC performs operation dispatch opens up an entirely new suite of deployment types and solves a host of problems common in today's cloud-native, distributed applications.

Put another way, a _lattice is a seamless, distributed unit of compute that can self-form atop any combination of cloud, edge, and physical infrastructure._
