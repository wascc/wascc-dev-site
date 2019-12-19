---
title: Create a Runtime Host
linktitle: Create Host
toc: true
type: docs
date: "2019-12-09T00:00:00Z"
draft: false
menu:
  first-actor:
#    parent: Example Topic
    weight: 4

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

To use **waSCC** to host actor modules, you can either use the generic binary that loads modules based on command line arguments, or you can create your own custom host with the [wascc-host](https://github.com/wascc/wascc-host) library. In this part of the tutorial, we'll create our own custom host so illustrate the waSCC library's API. 

## Create a New Binary Project
To create a new host, we don't need any scaffolding--we can just create a new empty binary Rust project:

```shell
$ cargo new hellorunner --bin
     Created binary (application) `hellorunner` package
```

## Use the waSCC Library
To build our demo, we'll need to add a few dependencies to our `Cargo.toml` file:

```
[dependencies]
env_logger = "0.7.1"
wascc-host = "(wascc version number)"
```

The first (`env_logger`) is not a requirement of waSCC, but we use it to print log info to output.

The second, `wascc-host`, is the main waSCC library for building hosts. Get the latest verson from [crates.io](https://crates.io/crates/wascc-host).

## Write Host Setup Code
The next step is to write our `main()` function. In this function we will initialize the host runtime with our actor module and a capability provider. Change your `main.rs` to the following:

```rust
use std::collections::HashMap;
use wascc_host::{host, Actor, NativeCapability};

fn main() -> std::result::Result<(), Box<dyn std::error::Error>> {
    env_logger::init();
    host::add_actor(Actor::from_file("../hellohttp/hello_signed.wasm")?)?;
    host::add_native_capability(NativeCapability::from_file(
        "../../wascc/wascc-host/examples/.assets/libwascc_httpsrv.so",
    )?)?;

    host::configure(
        "MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR",
        "wascc:http_server",
        generate_port_config(8081),
    )?;

    std :: thread :: park();

    Ok(())
}

fn generate_port_config(port: u16) -> HashMap<String, String> {
    let mut hm = HashMap::new();
    hm.insert("PORT".to_string(), port.to_string());

    hm
}
```

The first important line of code is:

```rust
host::add_actor(Actor::from_file("../hellohttp/hello_signed.wasm")?)?;
```

This will add a new actor module to the runtime host. A thread for handling this actor's messages will be immediately provisioned.

Next, we add a native capability provider plugin that will provide an implementation of the HTTP server. A pre-built Linux `.so` version is available [here](https://github.com/wascc/wascc-host/tree/master/examples/.assets). To run this on macOS, you will need to build `libwascc_httpsrv` as a `dylib` and point the path above to the `libwascc_httpsrv.dylib`. The fastest way to do this is to clone [this project](https://github.com/wascc/http-server-provider) on a Mac, and then do a `cargo build`. The dynamic library will be built automatically.

```rust
// Linux version:
host::add_native_capability(NativeCapability::from_file(
    "../../wascc/wascc-host/examples/.assets/libwascc_httpsrv.so",
)?)?;

// macOS version (you need to build the dylib yourself):
host::add_native_capability(NativeCapability::from_file(
    "path/to/libwascc_httpsrv.dylib",
)?)?;
```

In waSCC, both actors _and_ capability providers are _reactive_. This means that the capability provider won't do anything on behalf of an actor until it has been configured. So we send a `HashMap<String, String>` of configuration data to a particular capability ID (`wascc:http_server`) for a given actor's public key (the `MCY...` string). In this case, we tell the host that our actor's HTTP capability (`wascc:http`) should be configured to answer requests on port `8081`.

In the [Sign Module](sign_module.md) section, we generated a _module key_, and the output looked something like this:

```
Public Key: Mxxxxxxxxxxxxxxx
Seed: Sxxxxxxxxxx

Remember that the seed is private, treat it as a secret.
```

The _Public Key_ value is used to identify and validate the actor module. So copy the Public Key into the call to `host::configure`:

```rust
host::configure(
    "Mxxxxxxxxxxxxxxx",  // <-- Your actor's "module" public key goes here
    "wascc:http_server",
    generate_port_config(8081),
)?;
```

Once we've called `host::configure`, the HTTP server capability has spun up a new thread and has created a new server dedicated specifically to our actor module on the port number specified.

As you go through this tutorial on your own, make sure that the paths/filenames are appropriate for your environment. The location of your `libwascc_httpsrv.so` file (found by compiling the [waSCC HTTP Server Provider](https://github.com/wascc/http-server-provider)) may differ from the text of this tutorial.

Once you can compile this stage of the tutorial with a simple `cargo build`, move on to the next step.
