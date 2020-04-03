---
title: Build an In-memory Key-Value Store
linktitle: Build K/V Store
toc: true
type: docs
date: "2019-12-09T00:00:00Z"
draft: false
menu:
  native-provider:
#    parent: Example Topic
    weight: 3

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

In this section of the tutorial, we'll create some simple in-memory key-value store functionality and expose it through the capability provider interface.

## Modify the Boilerplate

The first thing we're going to want to do is customize the `src/lib.rs` file so that we're identifying as the right provider.
Change the `CAPABILITY_ID` const to `wascc:keyvalue` and change the return value of the `name()` function in the `CapabilityProvider` implementation of `KeyvalueProvider` to return something that reflects what we're building:

```rust
fn name(&self) -> &'static str {
        "waSCC Sample Key-Value Provider (In-Memory)"
}
```

## Create a Key-Value Store

The [waSCC actor SDK](https://github.com/wascc/wascc-actor) exposes a number of different functions for interacting with an abstract key-value store. If you take a look at the functions available in the `KeyValue` struct, you'll see that we support the following types of values:

* Atomic (a numeric value that has a concurrency-safe access pattern via increment/decrement)
* Scalar (a simple string)
* List (an order-preserved list of values)
* Set (a list with no duplicates and non-deterministic order)

To start, let's create a new module in `src/kv.rs` with the following code to get us going:

```rust
use std::collections::HashMap;
use std::collections::HashSet;
use std::error::Error;
use std::result::Result;

pub enum KeyValueItem {
    Atomic(i32),
    Scalar(String),
    List(Vec<String>),
    Set(HashSet<String>),
}

pub struct KeyValueStore {
    items: HashMap<String, KeyValueItem>,
}
```

Using this data structure, we're going to maintain a `HashMap` of our key-value data, with the keys being of type `String` and the values being variants of `KeyValueItem`. The module we're working on is essentially the implementation of the `KeyValueStore` struct. We want to provide functions that allow queries or data manipulations much the same way we would want if we had a Redis or Cassandra client.

The general access pattern for each function will be to create new hash map entries or look up the entry in the hash map and perform the appropriate manipulation on that entry.

Let's take a look at a subset of this struct's implementation:

```rust
impl KeyValueStore {
    pub fn new() -> Self {
        KeyValueStore {
            items: HashMap::new(),
        }
    }

    pub fn get(&self, key: &str) -> Result<String, Box<dyn Error>> {
        self.items.get(key).map_or_else(
            || Err("No such key".into()),
            |v| {
                if let KeyValueItem::Scalar(ref s) = v {
                    Ok(s.clone())
                } else {
                    Err("Attempt to fetch non-scalar".into())
                }
            },
        )
    }

    pub fn set(&mut self, key: &str, value: String) -> Result<(), Box<dyn Error>> {
        self.items
            .entry(key.to_string())
            .and_modify(|v| {
                if let KeyValueItem::Scalar(_) = v {
                    *v = KeyValueItem::Scalar(value.clone());
                }
            })
            .or_insert(KeyValueItem::Scalar(value));
        Ok(())
    }

    // ... addititional functions ...
}
```

Here we have the implementations of `get` and `set`. For `get`, we'll return an error if the key doesn't exist, or if the key does exist but it's not of type `KeyValueItem::Scalar`. For `set`, we will either overwrite the existing value at that key or we'll insert a new one.

There are a number of other functions to implement, and I will leave those as an exercise for the reader. If you want to see the completed module as a reference, you can find it [in the GitHub repository](https://github.com/wascc/examples/blob/master/keyvalue-provider/src/kv.rs).

If you're new to Rust, then this is a great opportunity to get some practice working on data structures and functional programming patterns. If this is all review for you, then feel free to copy what you need from the repository.

## Expose the K-V Store via the Capability Provider Interface

Capability providers are _loosely coupled_, and as such there's no direct connection between them and the actors that use them. Actors can't call the functions we just wrote in `src/kv.rs` directly. Instead, **messages** are **dispatched** from the actors to a capability provider through the waSCC host runtime. Think of what we're going to write now as a _message-based API facade_.

The sole entry point to this message-based API is the `handle_call` function. Here we branch on the _operation code_ we receive, execute the appropriate function, and return a result to the caller (the waSCC host).

If we were building a brand new capability, then we would want to create a "types" or a "codec" crate to allow us to share the `protobuf` definitions for messages and constants for the operation codes. Since we're working with a core capability already supported by waSCC, those definitions are available in the [wascc-codec](https://crates.io/crates/wascc-codec) crate.

Let's update our `handle_call` function (in `src/lib.rs`):

```rust
// Invoked by host runtime to allow an actor to make use of the capability
// All providers MUST handle the "configure" message, even if no work will be done
fn handle_call(&self, actor: &str, op: &str, msg: &[u8]) -> Result<Vec<u8>, Box<dyn Error>> {
    trace!("Received host call from {}, operation - {}", actor, op);

    match op {
        OP_BIND_ACTOR if actor == "system" => self.configure(msg.to_vec().as_ref()),
        OP_REMOVE_ACTOR if actor == "system" => {
            self.remove_actor(CapabilityConfiguration::decode(msg).unwrap())
        }
        keyvalue::OP_ADD => self.add(actor, AddRequest::decode(msg).unwrap()),
        keyvalue::OP_DEL => self.del(actor, DelRequest::decode(msg).unwrap()),
        keyvalue::OP_GET => self.get(actor, GetRequest::decode(msg).unwrap()),
        keyvalue::OP_CLEAR => self.list_clear(actor, ListClearRequest::decode(msg).unwrap()),
        keyvalue::OP_RANGE => self.list_range(actor, ListRangeRequest::decode(msg).unwrap()),
        keyvalue::OP_PUSH => self.list_push(actor, ListPushRequest::decode(msg).unwrap()),
        keyvalue::OP_SET => self.set(actor, SetRequest::decode(msg).unwrap()),
        keyvalue::OP_LIST_DEL => {
            self.list_del_item(actor, ListDelItemRequest::decode(msg).unwrap())
        }
        keyvalue::OP_SET_ADD => self.set_add(actor, SetAddRequest::decode(msg).unwrap()),
        keyvalue::OP_SET_REMOVE => {
            self.set_remove(actor, SetRemoveRequest::decode(msg).unwrap())
        }
        keyvalue::OP_SET_UNION => self.set_union(actor, SetUnionRequest::decode(msg).unwrap()),
        keyvalue::OP_SET_INTERSECT => {
            self.set_intersect(actor, SetIntersectionRequest::decode(msg).unwrap())
        }
        keyvalue::OP_SET_QUERY => self.set_query(actor, SetQueryRequest::decode(msg).unwrap()),
        keyvalue::OP_KEY_EXISTS => self.exists(actor, KeyExistsQuery::decode(msg).unwrap()),
        _ => Err("bad dispatch".into()),
    }
}
```

The starter template gave us an empty implementation for handling the `OP_BIND_ACTOR` operation, but the rest of these we'll have to add implementations for. Let's take a look at the code for handling `OP_SET` and `OP_GET`, which are just thin wrappers around the functionality in the `kv` module (in `src/lib.rs`):

```rust
impl KeyvalueProvider {
    // ... other functions ...

    fn get(&self, _actor: &str, req: GetRequest) -> Result<Vec<u8>, Box<dyn Error>> {
        let store = self.store.read().unwrap();
        if !store.exists(&req.key)? {
            Ok(bytes(GetResponse {
                value: String::from(""),
                exists: false,
            }))
        } else {
            let v = store.get(&req.key);
            Ok(bytes(match v {
                Ok(s) => GetResponse {
                    value: s,
                    exists: true,
                },
                Err(e) => {
                    eprint!("GET for {} failed: {}", &req.key, e);
                    GetResponse {
                        value: "".to_string(),
                        exists: false,
                    }
                }
            }))
        }
    }

    fn set(&self, _actor: &str, req: SetRequest) -> Result<Vec<u8>, Box<dyn Error>> {
        let mut store = self.store.write().unwrap();
        store.set(&req.key, req.value.clone())?;
        Ok(bytes(SetResponse { value: req.value }))
    }
}
```

There's an implied business rule in here that if a key doesn't exist, the `get` will return an empty string and an `Ok`. This is a smell for production because you lose the ability to distinguish between a non-existent key and a key that's holding an empty string. But, since we're safely in tutorial land, it won't cause us much friction.

In classic RPC fashion, the `get` function takers a `GetRequest` message, makes use of the key-value store module, and responds with a `GetResponse` message. The `set` function, along with all the others, follows the same pattern. I will leave the implementation of these facades up to the reader, but you can refer to the [finished example](https://github.com/wascc/examples/blob/master/keyvalue-provider/src/lib.rs) in GitHub if you like.
