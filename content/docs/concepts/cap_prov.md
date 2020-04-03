+++
title = "Capability Providers"
linktitle = "Capability Providers"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 15

[menu.docs]
  parent = "concepts"
  weight = 15
+++

A waSCC runtime is essentially a facilitator between _actors_ and _capabilities_. An actor is an encapsulation of one or more logical features, and in order to implement those features, actors make use of _capabilities_.

Every capability has a unique identifier. By convention, these capabilities have a _namespace prefix_. For example, all of the capabilities that are provided as part of the basic set that comes with the waSCC runtime have the namespace prefix of `wascc`, e.g. `wascc:messaging`, `wascc:keyvalue`, `wascc:http_server`, and `wascc:http_client`.

Capabilities, or _capability providers_, come in two forms: native and portable. Each is discussed in more detail on this page.

## Native Capabilities

A _native capability provider_ is a Linux shared object (`.so`) file. It is a _plugin_ that conforms to a specific C FFI contract. When a waSCC host runtime wishes to make a capability available for actors, it can load this native capability from disk.

Native capabilities have no restrictions on what they can and cannot do. They have the same privileges as the parent waSCC host runtime process. This means they can make unfettered network calls, spawn sub-processes, and generally take advantage of any kernel services they wish.

This is important, because it means that native capability providers can be multi-threaded and communicate with other services across the network. Given the current state of the WebAssembly community and the **WASI** standard, the vast majority of capability providers you encounter will be native providers.

As with all other capabilities, communication with actors is done through simple messaging. Messages are made up of a _binding_, an _operation_ and a _payload_.

Communication can be bi-directional--an actor may send messages to or make requests of a capability provider, and a capability can deliver messages to an actor.

## Portable Capabilities

A _portable capability_ is actually just an actor that has been given slightly more privilege within the waSCC host runtime. Rather than being loaded as a pure, isolated WebAssembly module (the `wasm32-unknown-unknown` target in Rust), portable capability providers are loaded as **[WASI](https://wasi.dev)** modules. This means they can make POSIX-like, low-level system calls but cannot do things like create new threads or otherwise exceed the boundaries of the WebAssembly specification.

Today, the number of realistic use cases for a portable capability is low due to the limitations of the WASI specification. However, as WASI gains stable networking constraints, then portable capability providers will automatically be able to benefit from those advances.

## Capability Support

waSCC supports both portable and native capabilities loaded into the same host runtime simultaneously. It will continue to support native capabilities until WASI has advanced far enough to allow the majority of capability providers to be written as high-privilege actors.

If you're interested in seeing how to create capability providers (both portable and native), then check out the [tutorials](/tutorials) section.
