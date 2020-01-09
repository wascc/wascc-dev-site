---
title: Swap Capability Provider Implementations
linktitle: Swap Providers
toc: true
type: docs
date: "2020-01-08T00:00:00Z"
draft: false
menu:
  native-provider:
#    parent: Example Topic
    weight: 5

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 5
---

Now that we've built our own capability provider, it's time to reap one of the biggest benefits of the decoupled sandbox provided by the waSCC host: the ability to _dynamically bind capability providers_.

To show how this works, we'll swap out the in-memory provider with the Redis provider (also available in the `.assets` folder) without having to recompile our actor.

Take a look at the new `src/main.rs` of our example:

```rust
use std::collections::HashMap;
use wascc_host::{host, Actor, NativeCapability};

fn main() -> std::result::Result<(), Box<dyn std::error::Error>> {
    env_logger::init();
    host::add_actor(
        Actor::from_file("../../wascc/wascc-host/examples/.assets/kvcounter.wasm")?)?;


    host::add_native_capability(NativeCapability::from_file(
        "../../wascc/wascc-host/examples/.assets/libwascc_httpsrv.so",
    )?)?;

    //host::add_native_capability(NativeCapability::from_file(
    //    "../../wascc/wascc-host/examples/.assets/libkeyvalue.so",
    //)?)?;
    host::add_native_capability(NativeCapability::from_file(
        "../../wascc/wascc-host/examples/.assets/libredis_provider.so",
    )?)?;

    host::configure(
        "MASCXFM4R6X63UD5MSCDZYCJNPBVSIU6RKMXUPXRKAOSBQ6UY3VT3NPZ",
        "wascc:http_server",
        generate_port_config(8081),
    )?;
    host::configure(
        "MASCXFM4R6X63UD5MSCDZYCJNPBVSIU6RKMXUPXRKAOSBQ6UY3VT3NPZ",
        "wascc:keyvalue",
        redis_config(),
    )?;

    std :: thread :: park();

    Ok(())
}

fn redis_config() -> HashMap<String, String> {
    let mut hm = HashMap::new();
    hm.insert("URL".to_string(), "redis://127.0.0.1:6379".to_string());

    hm
}

fn generate_port_config(port: u16) -> HashMap<String, String> {
    let mut hm = HashMap::new();
    hm.insert("PORT".to_string(), port.to_string());

    hm
}
```

The only changes were adding a `redis_config` function, adding another call to `configure` to supply the Redis configuration, and swapping the call to `add_native_capability` with the Redis library instead of our in-memory store.

Playing around with the multiple variants of this sample may give you some ideas as to how you can utilize the ability to swap out different implementations for the same capability ID. This is an incredibly useful feature for things like acceptance testing, supporting multiple clouds or cloud providers, and upgrading capabilities with no impact on running applications.

At this point you should be familiar with the process of creating actors and native capability providers. Hopefully you're getting some ideas for how you can put these features to work for your own development project.
