---
title: Creating an Actor Project in Rust
linktitle: Create an Actor Project
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

```shell
$ cargo install cargo-generate
```

Once you've got `cargo generate` installed, run it with the following command to create the new project:

```shell
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

actor_handlers! { codec::http::OP_HANDLE_REQUEST => hello_world,
                  codec::core::OP_HEALTH_REQUEST => health }

fn hello_world(_req: codec::core::http::Request) -> HandlerResult<codec::http::Response> {
    Ok(codec::http::Response::ok())
}

fn health(_req: codec::core::HealthRequest) -> HandlerResult<()> {
    Ok(())
}
```

There are only a couple of things you need to do as an actor developer to build fully functioning services. 

First, you'll need to invoke the `actor_handlers!` macro, which tells the actor API which _operations_ you are going to handle, and which functions will be called in response. The API automatically converts the messages into the appropriate data types for you.

`codec::http::OP_HANDLE_REQUEST` and `codec::core::OP_HEALTH_REQUEST` are constants defined by the actor SDK. As you get into more advanced tutorials, we'll walk through some patterns to define your own operation constants.

## Handle HTTP Requests

If you were to run this module inside the **waSCC** host runtime right now, it would simply return a `200 OK` response with no payload. Let's make a few changes to this module to make it do something a little more interesting.

We're going to return a simple JSON payload to the callers, so the first thing we want to do is add the `serde_json` crate to our dependencies. Add the following line (version number may be outdated) to your `Cargo.toml` file in the dependencies section:

```
serde_json = "1.0.50"
```

Next, add the following to the top of your `src/lib.rs` file to let us use the `json!` macro:

```rust
#[macro_use]
extern crate serde_json;
```

Now let's replace the `hello_world` function with the following code:

```rust
fn hello_world(_req: codec::http::Request) -> HandlerResult<codec::http::Response> {
    let result = json!({ "hello": "world", "data": 21});
    Ok(codec::http::Response::json(result, 200, "OK"))
}
```

When we run this in the host runtime, it will return this JSON payload in response to every HTTP request received. It's worth noting here that there's no code inside the actor that cares _how_ we're hosting HTTP, nor is there any code that is tightly coupled to a specific HTTP server implementation.

## Compiling

If you don't already have the `wasm32-unknown-unknown` Rust cross-compilation target installed, you can install it with the following command:

```shell
$ rustup target add wasm32-unknown-unknown
```

At this point we can compile the code with `cargo build`. This will build the WASM binary and place it in `target/wasm32-unknown-unknown/debug`.

In the next step, we'll cover how to sign an actor module, preparing it to run inside a **waSCC** host.
