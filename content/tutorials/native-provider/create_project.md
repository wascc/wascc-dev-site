---
title: Creating a Native Capability Provider Project in Rust
linktitle: Create Provider Project
toc: true
type: docs
date: "2020-01-08T00:00:00Z"
draft: false
menu:
  native-provider:
#    parent: Example Topic
    weight: 2

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---
During this part of the tutorial, you will create a new empty _capability provider_ project that will compile to a linux shared object (`.so`) library. We'll use this library as a capability provider plug-in for a **waSCC** host that we'll build at the end of the tutorial.

**NOTE** - This tutorial assumes that you're building on a **Linux** operating system. If you're using a Mac, your compilation targets (and filenames) will end in `.dylib` instead of `.so`. We have not tested this tutorial at all on **Windows**.

## Generate a New Project

While you can create your own Rust project from scratch, it's far easier to use the pre-supplied starter template. Using `cargo generate`, you can create a new project that is already configured to build a native _capability provider_.

If you completed the first tutorial, you should already have `cargo generate` installed. Run the following command to create a new project:

```shell
$ cargo generate --git https://github.com/wascc/new-provider-template
 Project Name: keyvalue
 Creating project called `keyvalue`...
 Done! New project created /home/kevin/Code/testing/keyvalue
```

You can choose whatever project name you like, but for this example we used `keyvalue`.

## Examine the Starter Code

Once you've created your new project, you will have the following code in your `src/lib.rs` file:

```rust
#[macro_use]
extern crate wascc_codec as codec;

#[macro_use]
extern crate log;

use codec::capabilities::{CapabilityProvider, Dispatcher, NullDispatcher};
use codec::core::OP_CONFIGURE;
use wascc_codec::core::CapabilityConfiguration;

use std::error::Error;
use std::sync::RwLock;

capability_provider!(KeyvalueProvider, KeyvalueProvider::new);

const CAPABILITY_ID: &str = "new:keyvalue"; // TODO: change this to an appropriate capability ID

pub struct KeyvalueProvider {
    dispatcher: RwLock<Box<dyn Dispatcher>>,
}

impl Default for KeyvalueProvider {
    fn default() -> Self {
        env_logger::init();

        KeyvalueProvider { 
            dispatcher: RwLock::new(Box::new(NullDispatcher::new())),
        }
    }
}

impl KeyvalueProvider {
    pub fn new() -> Self {
        Self::default()
    }

    fn configure(
        &self,
        config: impl Into<CapabilityConfiguration>,
    ) -> Result<Vec<u8>, Box<dyn Error>> {
        let _config = config.into();

        Ok(vec![])
    }
}

impl CapabilityProvider for KeyvalueProvider {
    fn capability_id(&self) -> &'static str {
        CAPABILITY_ID
    }

    // Invoked by the runtime host to give this provider plugin the ability to communicate
    // with actors
    fn configure_dispatch(&self, dispatcher: Box<dyn Dispatcher>) -> Result<(), Box<dyn Error>> {
        trace!("Dispatcher received.");
        let mut lock = self.dispatcher.write().unwrap();
        *lock = dispatcher;

        Ok(())
    }

    fn name(&self) -> &'static str {
        "New Keyvalue Capability Provider" // TODO: change this friendly name
    }

    // Invoked by host runtime to allow an actor to make use of the capability
    // All providers MUST handle the "configure" message, even if no work will be done
    fn handle_call(&self, actor: &str, op: &str, msg: &[u8]) -> Result<Vec<u8>, Box<dyn Error>> {
        trace!("Received host call from {}, operation - {}", actor, op);

        match op {
            OP_CONFIGURE if actor == "system" => self.configure(msg.to_vec().as_ref()),
            _ => Err("bad dispatch".into()),
        }
    }
}
```

## Implement the Capability Provider Interface

Every native capability provider must implement the capability provider interface. This means implementing four functions:

* `capability_id` - This function returns the capability ID of the capability provider. This is _not_ a globally unique identifier, but an identifier of the _capability abstraction_ for which this library is a provider. For example, we'll be using `wascc:keyvalue` in the rest of this tutorial to provide an in-memory key-value store and we'll see that the Redis key-value store also uses that same capability ID. **NOTE** that only 1 capability provider per capability ID can be loaded into the same host runtime.
* `name` - This function returns a human-readable (log-friendly) name of the capability provider that should also include the name of the specific implementation. For example, a Redis key-value provider might return the string `"Key-Value Provider (Redis)"`
* `configure_dispatch` - Each capability provider will handle this function call _once_ during its lifetime. A dispatcher is supplied to each provider to give it a means with which it can communicate with the host runtime (and, by extension, that runtime's hosted actors)
* `handle_call` - If an actor sends a message to a capability provider to make use of that capability, that message will arrive via this function.

## Compiling

At this point we can compile the code with `cargo build`. This will build the library file in `target/debug` called `libkeyvalue.so` (or `libkeyvalue.dylib` on a Mac).

In the next step, we'll go through the process of implementing an in-memory key-value store in Rust and exposing that functionality through the capability provider interface.
