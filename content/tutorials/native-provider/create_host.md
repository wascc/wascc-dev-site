---
title: Create a Runtime Host
linktitle: Create Host
toc: true
type: docs
date: "2019-12-09T00:00:00Z"
draft: false
menu:
  native-provider:
#    parent: Example Topic
    weight: 4

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

To use **waSCC** to host an actor module that makes use of our key-value store, we'll create a new binary project using the [wascc-host](https://crates.io/crates/wascc-host) crate.

## Create a New Binary Project

To create a new host, we don't need any scaffolding--we can just create a new empty binary Rust project:

```shell
$ cargo new kvrunner --bin
     Created binary (application) `kvrunner` package
```

## Use the waSCC Library

To build our demo, we'll need to add a few dependencies to our `Cargo.toml` file:

```
[dependencies]
env_logger = "0.7.1"
wascc-host = "0.6.0" # Check to make sure you're using the latest version
```

The first (`env_logger`) is not a requirement of waSCC, but we use it to print log info to output.

The second, `wascc-host`, is the main waSCC library for building hosts. Get the latest verson from [crates.io](https://crates.io/crates/wascc-host).

## Write Host Setup Code

The next step is to write our `main()` function. In this function we will initialize the host runtime with our actor module, an HTTP capability provider, and our newly written in-memory K/V capability provider. Change your `main.rs` to the following:

```rust
use std::collections::HashMap;
use wascc_host::{host, Actor, NativeCapability};

fn main() -> std::result::Result<(), Box<dyn std::error::Error>> {
    env_logger::init();
    host::add_actor(
        Actor::from_file("../../wascc/wascc-host/examples/.assets/kvcounter.wasm")?)?;

    host::add_native_capability(NativeCapability::from_file(
        "../../wascc/wascc-host/examples/.assets/libwascc_httpsrv.so",
        None,
    )?)?;

    // Add the in-memory key-value provider
    host::add_native_capability(NativeCapability::from_file(
        "../../wascc/wascc-host/examples/.assets/libkeyvalue.so",
        None,
    )?)?;

    host::bind_actor(
        "MASCXFM4R6X63UD5MSCDZYCJNPBVSIU6RKMXUPXRKAOSBQ6UY3VT3NPZ",
        "wascc:http_server",
        None,
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
host::add_actor(
    Actor::from_file("../../wascc/wascc-host/examples/.assets/kvcounter.wasm")?)?;
```

This will add a new actor module to the runtime host. You can find this file in the [wascc-host](https://github.com/wascc/wascc-host/tree/master/examples/.assets) examples GitHub, along with the other files necessary to complete this tutorial.

Next, we add a native capability provider plugin that will provide an implementation of the HTTP server. As with the [first tutorial](/tutorials/first-actor), you'll need to build your own `dylib` file if you're on a Mac. Otherwise, you can use the pre-built Linux assets.

```rust
// Linux version:
host::add_native_capability(NativeCapability::from_file(
    "../../wascc/wascc-host/examples/.assets/libwascc_httpsrv.so",
    None,
)?)?;

// macOS version (you need to build the dylib yourself):
host::add_native_capability(NativeCapability::from_file(
    "path/to/libwascc_httpsrv.dylib",
    None,
)?)?;
```

Right now we can get away without calling `bind_actor` on the `wascc:keyvalue` capability because we're using the in-memory provider. Go ahead and run
the binary (from the project root):

```shell
$ RUST_LOG=info,cranelift_wasm=warn cargo run
```

Now when you issue curl commands from the shell, you'll get count values:

```shell
$ curl localhost:8081/mycount | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    14  100    14    0     0  14000      0 --:--:-- --:--:-- --:--:-- 14000
{
  "counter": 12
}
```

Remember that since we're using a transient provider, if you shut down and re-start the waSCC host process, the counter will reset to zero.

In the next step, we'll see how easy it is to switch between key-value providers without recompiling our actor.
