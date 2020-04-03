---
title: "waSCC 0.6.0 Release Notes"
subtitle: "Named Bindings, WasccHost, and a new Actor API"
summary: "This release is another release that brakes a lot of things from previous versions. The main features we added are named bindings, a new host API, and a new actor API."
authors: ["kevin"]
tags: []
categories: ["release-notes"]
date: 2020-04-03T10:14:33-04:00
lastmod: 2020-04-03T10:14:33-04:00
featured: false
toc: true
draft: false
projects: []
---
In this release, we basically broke _all the things_. When building software, especially open source projects that have existing consumers, some of which we don't even know about, the decision to do breaking changes is harder than it looks.

## Named Bindings

This was a change that started at the [waPC](https://github.com/wapc) level with the addition of the `binding` field to the `host_call` function in the RPC specification. In previous versions of waPC (and, transitively, waSCC), when a guest module wanted to make a host call, it could do so by specifying an `operation` and a `namespace`. In the waSCC model, the `namespace` is the _capability ID_ of a given provider, e.g. `wascc:messaging`.

For a while this seemed like it would be enough, but a number of compelling use cases weren't possible. What if we wanted to allow _multiple instances_ of an RPC target to exist within the same namespace in the host? In waSCC terms, this meant things like loading multiple `wascc:keyvalue` providers into a single host. Without the new `binding` field, the actor has no way of differentiating between these instances.

I think the most common use case that explains this feature is having multiple key-value stores. You might be building an actor that has a key-value store it needs for long-term storage, maybe for something like a user profile. It then might also have a key-value store it's using for short-term storage like a cache or other transient values.

You could have a database like Redis provide both of these features, and differentiate between them with different connection strings. But, you might want Redis for one and Cassandra for the other, or Redis and Etcd, and so on.

With waSCC 0.6.0, this is now possible. When you load a capability provider, you specify a _binding name_ (or leave it off, in which case it becomes "default"). In our hypothetical example, you could load one provider for the `userprofile` binding and another one for the `cache` binding.

It's important to note that these are just names, and they imply a _logical meaning_ to the provider in the binding. Actors still remain loosely coupled from the capability providers. As you'll see in the new actor API, you can specify the binding names when you request certain host capabilities.

## The WasccHost API

Since the first version of waSCC, the API has been through functions floating loosely within the `host` module. This way, you would call functions like `host::add_actor`. The implicit fact in this is that there's a number of _global statics_ hovering inside this module.

While there are still a few of them (I am working on reducing them, but with FFI callbacks it's difficult), there is now a single `WasccHost` struct that is the developer-facing API to the waSCC host runtime now. All of the same functions are there, though some have been renamed. 

To create a new host, we now have to call `WasccHost::new()`, and then invoke methods on the new instance like `host.add_actor(...)`. The old `configure()` function has been renamed to `bind_actor` to more clearly describe that what's happening during that call is applying a set of configuration values to an _actor binding_, a connection between an actor and a capability provider.

## The Actor API

The Rust actor API has been undergoing rapid change lately, with a number of improvements to overall ergonomics that I think make it extremely easy to use, even if Rust isn't your first language. The forthcoming AssemblyScript API will have a similar look and feel, though it will be idiomatic AssemblyScript.

In this new Rust API, we no longer have a `CapabilitiesContext` that needs to be passed around everywhere that we might want host functionality. Now we can create an instance of a wrapper around host capabilities wherever it is needed, and not before then. Here's an example of the new "Key-Value Counter" example that illustrates handling an HTTP request, incrementing values with the key-value provider, and responding with an HTTP response:

```
extern crate wascc_actor as actor;

#[macro_use]
extern crate serde_json;

use actor::prelude::*;

actor_handlers! { codec::http::OP_HANDLE_REQUEST => increment_counter,
                  codec::core::OP_HEALTH_REQUEST => health }

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

This is a remarkably small amount of code to write compared to the functionality that you're getting. The new way of interacting with capabilities involves accessing the capability and a binding (or `default` if you don't need a named one), e.g. `keyvalue::default::atomic_add(...)?;`)

## Miscellaneous

In this release we also caught a few minor bugs, updated and improved some of the code documentation, and got to remove some technical debt. The internal dispatch plumbing, as well as the thread handlers for actors and capabilities, were convoluted and difficult to read and modify. The named bindings feature was an excellent excuse to get back into that section of the code and clean it up.

Looking forward to seeing what you're all going to build with this new version of waSCC!