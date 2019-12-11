+++
title = "WebAssembly Procedure Calls"
linktitle = "waPC"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 5

[menu.docs]
  parent = "concepts"
  weight = 5
+++

WebAssembly's current specification only allows for the host to invoke functions (exports) with numeric parameters and accept only a single numeric parameter in return. The same goes for the reverse of that communication--a WebAssembly module can only make host invocations (imports) with numeric parameters and a single numeric return value.

There are a number of ways to get around this. We could use tools and specifications to _generate_ strongly-typed wrappers for each of the modules that we want to host. There's quite a bit of merit to ideas like this. However, with **waPC**, _WebAssembly Procedure Calls_, we wanted the same kind of flexibility that you get with [gRPC](https://grpc.io/), where you can pass a formatted string as an operation, some arbitrary payload, and get some arbitrary payload in response.

This is how [waPC](https://github.com/wapc) was born. At its core, waPC is a contract that defines a means for bi-directional communication between guest and host where _neither the guest nor the host need know about the other's memory management strategy_. When we use tools that generate code based on some assumptions about the host runtime (like the ones that work with JavaScript), we generate a tight coupling to the memory management system of the host runtime.

waPC does no such thing and, giving the ability for wasm targets produced by multiple different languages to all easily comply with the contract. For example, we have built sample guests and hosts using **Zig**, a language that uses explicit memory allocators.
