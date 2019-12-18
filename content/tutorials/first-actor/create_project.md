---
title: Creating an Actor Project in Rust
linktitle: Create Actor Project
toc: true
type: docs
date: "2019-12-09T00:00:00Z"
draft: false
menu:
  first-actor:
#    parent: Example Topic
    weight: 2

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---
During this part of the tutorial, you will create a new empty _actor module_ WebAssembly project and add code to handle incoming HTTP requests.

## Cargo Generate New Project
While you can create your own Rust project from scratch, it's far easier to use the pre-supplied starter template. Using `cargo generate`, you can create a new project that is already configured to build an _actor module_.

If you don't already have `cargo generate` installed, you can install it with the following command:

```terminal
$ cargo install cargo-generate
```

Once you've got `cargo generate` installed, run it with the following command to create the new project:

```terminal
$ cargo generate --git https://github.com/wascc/new-actor-template
 Project Name: hellohttp
 Creating project called `hellohttp`...
 Done! New project created /home/kevin/Code/testing/hellohttp
```

You can choose whatever project name you like, but for this example we used `hellohttp`.

## Examine the Starter Code
Once you've created your new project, you will have the following code in your `src/lib.rs` file:

```rust
extern crate wascc_actor as actor;

use actor::prelude::*;

actor_receive!(receive);

pub fn receive(ctx: &CapabilitiesContext, operation: &str, msg: &[u8]) -> CallResult {    
    match operation {
        http::OP_HANDLE_REQUEST => hello_world(ctx, msg),
        core::OP_HEALTH_REQUEST => Ok(vec![]),
        _ => Err("Unknown operation".into()),
    }
}

fn hello_world(
   _ctx: &CapabilitiesContext,
   _payload: impl Into<http::Request>) -> CallResult {
    Ok(protobytes(http::Response::ok())?)
}
```

There are only a couple of things you need to do as an actor developer to build fully functioning services. 

First, you'll need to invoke the `actor_receive!` macro, which tells the actor SDK which function you're going to use to handle messages coming from the host runtime. The `http::OP_HANDLE_REQUEST` and `core::OP_HEALTH_REQUEST` are constants defined by the actor SDK. As you get into more advanced tutorials, we'll walk through some patterns to define your own operation constants.

## Handle HTTP Requests
If you were to run this module inside the **waSCC** host runtime right now, it would simply return a `200 OK` response with no payload. Let's make a few changes to this module to make it do something a little more interesting.

We're going to return a simple JSON payload to the callers, so the first thing we want to do is add the `serde_json` crate to our dependencies. Add the following line (version number may be outdated) to your `Cargo.toml` file in the dependencies section:

```
serde_json = "1.0.44"
```

Next, add the following to the top of your `src/lib.rs` file to let us use the `json!` macro:

```rust
#[macro_use]
extern crate serde_json;
```

Now let's replace the `hello_world` function with the following code:

```rust
fn hello_world(_ctx: &CapabilitiesContext, 
                 _payload: impl Into<http::Request>) -> CallResult {
    let result = json!({ "hello": "world", "data": 21});
    Ok(protobytes(http::Response::json(result, 200, "OK"))?)
}
```

When we run this in the host runtime, it will return this JSON payload in response to every HTTP request received. It's worth noting here that there's no code inside the actor that cares _how_ we're hosting HTTP, nor is there any code that is tightly coupled to a specific HTTP server implementation.

## Compiling

At this point we can compile the code with `cargo build`. This will build the WASM binary and place it in `target/wasm32-unknown-unknown/debug`.

In the next step, we'll cover how to sign an actor module, preparing it to run inside a **waSCC** host.
