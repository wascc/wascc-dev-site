+++
title = "Actors"
linktitle = "Actors"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 35

[menu.docs]
  parent = "concepts"
  weight = 10
+++

This document provides an overview of _actors_ as they apply to **wasCC** and WebAssembly.

# The Actor Model

The [actor model](https://en.wikipedia.org/wiki/Actor_model) has been around since **1979** and is not a new concept, though lately it seems to be enjoying something of a renaissance. An _actor_ is the fundamental primitive of concurrent computation.

Actors are _reactive_. In response to receiving a message, a traditional actor can:

* make local decisions
* create more actors
* send more messages
* determine how to respond to the next message received

In the case of a **waSCC** actor, the only activity that actors are not allowed to perform is _create new actors_, which is a secure, privileged activity reserved for the host runtime.

# WebAssembly Actors

A subtle, yet vitally important benefit of the actor model is that all of the code that you execute within the confines of the actor is _single-threaded_. Actor developers do not have to worry about multi-threading, concurrency, or re-entrant functions.

The job of the actor developer is to simply define a response to each type of message in which the actor is interested. Actors are also completely and blissfully unaware of how their non-functional requirements are being met.

Actors do not know if there is an HTTP server present, or if there is a message broker present. If they are present, the actors do not know _which_ specific products or services are proving those capabilities.

In _waSCC_, an actor defines **message handlers**, which the host runtime will invoke every time there is a new message targeted at that actor.

# Actors in Rust

In the [Rust actor SDK](https://github.com/wascc/wascc-actor), you indicate your handler functions to the host runtime by using the `actor_handlers!` macro:

```rust
actor_handlers { codec::http::OP_HANDLE_REQUEST => increment_counter,
                 codec::core::OP_HEALTH_REQUEST => health }
```

Then you simply define your handler functions, which can access host capabilities dynamically bound at runtime:

```rust
fn increment_counter(msg: codec::http::Request) -> CallResult {
    let key = msg.path.replace('/', ":");
    let value = keyvalue::default().atomic_add(&key, 1)?;
    
    let result = json!({ "counter": value });
    Ok(serialize(codec::http::Response::json(result, 200, "OK"))?)
}

fn health(_h: codec::core::HealthRequest) -> ReceiveResult {
    Ok(vec![])
}
```

Compiling this code into the `wasm32-unknown-unknown` target and signing it is then all that's necessary to build an actor. For more scenario-specific information on building actors, check out the [tutorials](/tutorials) section or the [GitHub examples](https://github.com/wascc/examples).
