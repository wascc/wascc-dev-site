+++
title = "How it Works"
linktitle = "How it Works"
date = 2020-06-10

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 98

[menu.docs]
  parent = "lattice"
  weight = 4
+++

Lattice is made possible by adding a distributed message bus (supported by [NATS](https://nats.io) servers and leaf nodes) to waSCC host runtime processes.

## Traditional Microservice Deployment

Take a look at this diagram of a traditional microservice development.

{{< figure library="true" src="lattice/fig1.png" title="Traditional Microservices Deployment" numbered="true" lightbox="true" >}}

Here you can see the 1:1 tight coupling of host process and service business logic, which is then usually wrapped in something like a docker container and then deployed on top of some kind of container scheduling environment like Kubernetes. In this development paradigm, microservices are tightly coupled to their host processes, which are in turn often not environment-agnostic. We can write and use all kinds of automation tools to reshape what we deploy to production, but once it's deployed, it's usually _static_ except for the usual scale-up and scale-down knobs we get from infrastructure.

## A Lattice of One

{{< figure library="true" src="lattice/fig2.png" title="Single Node Lattice" numbered="true" lightbox="true" >}}

A waSCC host (which can be thought of as a _single-node lattice_) contains an arbitrary number of actors and capability providers, immediately achieving a greater service/compute density than the traditional microservice development model. These actors are lightweight (often less than **1MB**), securely signed, and can run anywhere there's a suitable host.

## A Simple Lattice

{{< figure library="true" src="lattice/fig3.png" title="Simple Lattice" numbered="true" lightbox="true" >}}

A simple lattice is a flat group of waSCC hosts that have established a message bus connection, allowing actors and capability providers to bind and communicate _regardless of the parent process in which they reside_, and completely independent of the infrastructure substrate. It's important to emphasize the fact that actors and capability providers can move between host processes and the actor code never has to be recompiled or redeployed--all of this is dynamic, loosely coupled, and managed at runtime. The ability to _hot swap_ or _live update_ an actor live without dropping a single request is also supported in a lattice. You can remotely trigger an update to any actor, including multiple instances of the same actor.

## A Global Lattice (Seamless Environment Bridging)

{{< figure library="true" src="lattice/fig4.png" title="Bridged Lattice" numbered="true" lightbox="true" >}}

Through the power and flexibility of [NATS Leaf Nodes](https://docs.nats.io/nats-server/configuration/leafnodes), simple lattices can be stitched together to form larger, seamlessly connected lattices across an arbitrary number of bridge points. This means that you can create isolated zones of traffic within your own cloud and use a leaf node to bridge specific actor-provider calls into a different section of your infrastructure, to another cloud entirely, _and even to devices deployed in the field_.

## Runtime Relocation

The power of a waSCC lattice truly shines when you take advantage of remote administration capabilities. While a lattice remains connected (which, as previously mentioned, could be an arbitrary number of connected sub-lattices), you can issue commands to update an actor's `wasm` module to a new version, you can dynamically scale up or down the number of running actors and providers, and you can even choose to move an actor from one location to another to optimize its performance or traffic usage.

## It Should Just Work

The creators of waSCC have been building (and suffering through) distributed systems for nearly as long as systems have been distributed. We've seen the current trend toward adding layer after layer of complexity on top of frameworks and products, all allegedly in the name of making things easier to use. We want to achieve simplicity by stripping away layers of complexity. We don't want to have to write (or automate the creation of) dozens or hundreds of `yaml` files, we don't want every developer on our teams to have to be a certified expert in 12 different cloud infrastructure technologies. We want everything to _just work_.

waSCC lattices were born of the simple desire to want to stand up a fast, lightweight process that can _automatically join_ a flat, seamless cluster of connected processes. It should work by default and allow customization as needed _without adding friction_ to the process. We want to be able to start it up on our laptop and have it work while our network adapter is in airplane mode, we want it to work in our virtual machine test labs, in our QA clouds, in production, on physical on-premise devices, on tiny devices in the field, and anywhere else we can imagine.
