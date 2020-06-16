---
title: "Building a Remote Controlled Relay with waSCC and Lattice"
subtitle: "Fun with waSCC Lattice Mode, a Raspberry Pi, and a 4-Channel Relay HAT"
summary: "In this post we'll walk you through how to build a hardware-specific capability provider and how to talk to that provider remotely, from anywhere in the world, using nothing but waSCC actors, providers, and the new lattice mode"
authors: ["kevin"]
tags: ["lattice", "gpio", "rpi"]
categories: ["IoT"]
date: 2020-06-16T00:14:33-04:00
lastmod: 2020-06-16T00:14:33-04:00
featured: false
toc: true
draft: false
projects: []
---


## The Project

With the availability of the new [lattice](/docs/lattice/overview) distributed message bus feature, waSCC hosts can now communicate with each other anywhere, regardless of the supporting infrastructure[^1]. In an effort to load and acceptance test this while also having a little bit of fun, I decided to experiment with a project. I wanted to make an **"office busy status light"**--a lamp that I can turn on at will to indicate to my family that I'm in the middle of a conference. This would let them know not to come into the office unless the house or a child is on fire.

## Why Lattice

What makes lattice so powerful is that it _just works_. It doesn't need YAML files or a bunch of ceremony. It was designed with my hatred of complexity in mind. The core tenet of lattice is that _actors, hosts, and capability providers can be anywhere_. Actors and capabilities can be bound to each other in-process, out-of-process on the same network, or out-of-process across any number of routers, switches, and network hops.

This means that we can deploy a capability provider that only works in the presence of a set of [GPIO](https://en.wikipedia.org/wiki/General-purpose_input/output) pins on a Raspberry Pi to the device, and we can deploy an HTTP server and the supporting actor to the cloud. In the spirit of the future of distributed computing and today's buzzword affinity for _edge computing_, we can do what I call _adjacency-balanced computing_, and position each workload exactly where it needs to be for optimal use.

## Building a GPIO Relay Capability Provider

If we want to turn a lamp on or off, the easiest thing to do is control the lamp's power source[^2]. Controlling the power source of a component is done with a [relay](https://www.explainthatstuff.com/howrelayswork.html). In my case, I got a _Relay HAT_[^3], which is a nice 4-channel relay that slips onto the Raspberry Pi's GPIO pins without requiring any soldering or fussing with weak and failure-prone jumper wires.

I used the **[rppal](https://crates.io/crates/rppal)** crate to write the code that manipulates the four relays on my Raspberry Pi. You can find all of the code for the rest of this blog post  available in my **[officelight](https://github.com/autodidaddict/officelight)** repository.

Controlling a relay boils down to setting the voltage on a given pin to **HIGH** when you want power to the appliance to be available and setting it to be **LOW** when you want to shut the appliance off (or, in some cases, the reverse, depending on the relay config). Following the advice in the [creating a native provider](/tutorials/native-provider) tutorial, I created a basic provider with the following descriptor to describe the contract by which the provider and its bound actors must abide:

```rust
fn get_descriptor(&self) -> Result<Vec<u8>, Box<dyn Error>> {
    Ok(serialize(
        CapabilityDescriptor::builder()
            .id(CAPABILITY_ID) // gpio:relay
            .name("Raspberry Pi GPIO Relay") 
            .long_description(
                "A capability provider that exposes on/off functionality for GPIO-attached relays on the Raspberry Pi")
            .version(VERSION)
            .revision(REVISION)
            .with_operation(
                OP_DISABLE,
                OperationDirection::ToProvider,
                "Turns off the given relay",
            )
            .with_operation(
                OP_ENABLE, 
                OperationDirection::ToProvider, 
                "Turns on the given relay")
            .with_operation(
                OP_QUERY, 
                OperationDirection::ToProvider, 
            "Queries the status of all relays")
            .build(),
    )?)
}
```

Here's the simple provider code for toggling the relays[^4] (note that this provider doesn't return any data in response to toggle commands):

```rust
fn enable_relay(&self, relay: RelayIdentifier) -> Result<Vec<u8>, Box<dyn Error>> {
    let idx: usize = relay.relay_number as usize - 1;
    self.pins.write().unwrap()[idx].set_high();

    let mut lock = self.states.write().unwrap();
    if let Some(state) = lock.relay_states.get_mut(idx) {
        *state = true;
    }
    Ok(vec![])
}

fn disable_relay(&self, relay: RelayIdentifier) -> Result<Vec<u8>, Box<dyn Error>> {
    let idx: usize = relay.relay_number as usize - 1;
    self.pins.write().unwrap()[idx].set_low();

    let mut lock = self.states.write().unwrap();
    if let Some(state) = lock.relay_states.get_mut(idx) {
        *state = false;
    }

    Ok(vec![])
}
```

## Building the Office Busy Light Actor

As it should always be, our business logic is simple to write. We want our system to listen to incoming HTTP requests according to the following API:

* `GET` - Return the status of all the relays
* `PUT /relays/{1-based idx}` - Ensure that the relay is on (idempotent)
* `DELETE /relays/{1-based idx}` - Ensure that the relay is off (idempotent)

This is the full, unabridged code for the entire actor:

```rust
extern crate wascc_actor as actor;

#[macro_use]
extern crate serde;

use actor::prelude::*;
mod relay;

actor_handlers! {
    codec::http::OP_HANDLE_REQUEST => relay_service,
    codec::core::OP_HEALTH_REQUEST => health
}

fn relay_service(payload: codec::http::Request) -> HandlerResult<codec::http::Response> {
    match payload.method.as_ref() {
        "GET" => query_relays(),
        "PUT" | "DELETE" => set_relay(&payload.method, &payload.path),
        _ => Err("bad method".into()),
    }
}

fn health(_req: codec::core::HealthRequest) -> HandlerResult<()> {
    Ok(())
}

fn query_relays() -> HandlerResult<codec::http::Response> {
    let state = relay::default().query_relays()?;
    Ok(codec::http::Response::json(state, 200, "OK"))
}

fn set_relay(method: &str, path: &str) -> HandlerResult<codec::http::Response> {    
    if path.starts_with("/relays/") {
        let p = path[1..].to_string();
        let toks: Vec<&str> = p.split('/').collect();
        let relay_num: u8 = toks[toks.len() - 1].parse()?;
        if method == "DELETE" {
            relay::default().disable_relay(relay_num)?;
        } else {
            relay::default().enable_relay(relay_num)?;
        }
        Ok(codec::http::Response::ok())
    } else {
        Ok(codec::http::Response::bad_request())
    }
}
```

In the above code, the call to `relay::default().disable_relay(relay_num)` is just a call to a wrapper around the untyped host binding invocation. This wrapper pattern is the same way we expose provider APIs for things like messaging, the key-value store, etc:

```rust
pub fn enable_relay(&self, relay_num: u8) -> Result<()> {
    let id = gen_identifier(relay_num);
    let _ = untyped::host(&self.binding).call(CAPID_RELAY, OP_ENABLE, serialize(id)?)?;
    Ok(())
}
```

What is vitally important, and one of the most powerful aspects of **lattice**, is the code that you _do not see_ in the preceding sample.

The call to the capability provider is **blissfully unaware of where or how far away that provider may be**. It could be in-process, it could be out-of-process but on the same machine, or in our case, it could be on a remote Raspberry Pi behind a firewall and a managed switch. And as usual, the actor is also blissfully unaware of _which_ provider is satisfying the `gpio:relay` contract. It could be a mock, it could be an industrial relay, or it could be our Raspberry Pi.

## Running the Example

Most of this is fairly easy to set up and run. You can use the manifest files included in the Github repository to start up the HTTP server provider and the actor in one process using the generic (at least 0.9.0) `wascc-host` binary.

There's also a custom binary called `relayprovider` that pretends to be a waSCC host by listening on the right lattice subjects on NATS and forwarding those calls to the capability provider. I wrote that shim because the build pipelines available to me at the time were not able to deal with the OpenSSL transitive dependency on the `aarch64` target. Rather than mess with the pipeline, I just made a smaller binary and compiled it on the Raspberry Pi directly (running 64-bit Ubuntu).

WebAssembly is supposed to save us from all of these cross-platform building problems!

Here's what you need to run my "office busy light" sample:

* **waSCC Host Process** - HTTP Server Capability Provider and the Relay Actor
* **wasCC Host Process** (or custom shim) - GPIO Relay Capability Provider running on a Raspberry Pi with the 4-channel Relay HAT.
* **NATS** Server to which the previous two processes can connect

With these things in place, you'll be able to remotely control up to 4 appliances using waSCC, actors, and lattice.

_Go forth and tinker!_

{{< figure src="/pi/fig1.jpg" library="true" title="Status Light - Above Office Secret Door Bookshelf" numbered="false" lightbox="true" >}}

[^1]: This is due largely to the power and simplicity of [NATS](https://nats.io)
[^2]: I've made many attempts at making my own lamp with various LED combinations and the frailty and complexity of the hardware involved left me looking for a simpler solution.
[^3]: The one I used for this project I bought from [Amazon](https://www.amazon.com/KEYESTUDIO-4-Channel-Shield-Expansion-Raspberry/dp/B072XGF4Z3/ref=sr_1_2_sspa?dchild=1&keywords=raspberry+pi+relay+hat&qid=1592318485&sr=8-2-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyT05SMUY2N0s5SEs5JmVuY3J5cHRlZElkPUEwODg0MjA3VDVPQUVKQU5TNzBXJmVuY3J5cHRlZEFkSWQ9QTA2NjQ5OTQzSVlMMEFET1ZVRTBDJndpZGdldE5hbWU9c3BfYXRmJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), but you can find Pi Relay HATs anywhere and usually for a low price.
[^4]: For this example, I maintained my own booleans rather than detecting voltage on the pins. I found that occasionally checking the voltage would drop it to zero and shut off the relay. If this were a professional IoT product, I would obviously find a solution to check the voltage for real rather than risk out-of-sync state.
