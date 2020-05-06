---
title: "Creating Actors in AssemblyScript"
subtitle: "Developing Actors for the waSCC runtime with AssemblyScript"
summary: "This post takes a look at the AssemblyScript language, what it is, and why it's a powerful choice as one of the languages supported to build waSCC actors"
authors: ["kevin"]
tags: ["assemblyscript", "actors"]
categories: []
date: 2020-04-29T00:14:33-04:00
lastmod: 2020-04-29T00:14:33-04:00
featured: false
toc: true
draft: false
projects: []
---

## Introducing AssemblyScript

[AssemblyScript](https://docs.assemblyscript.org/) is a **strict subset** of [TypeScript](https://www.typescriptlang.org/) that compiles to **WebAssembly**. It is immediately approachable to a large number of developers because of its core foundation in JavaScript. However, it is **not** a _Node.js_ project, nor should it ever be thought of as "a TypeScript to WebAssembly compiler."

It's best to think of AssemblyScript as a fit-for-purpose, JavaScript-inspired, strongly-typed WebAssembly language.

This language compiles to the WebAssembly binary format, and _only that format_, producing `.wasm` files. It adheres to all of the standards and specifications, which cannot be said of some other languages. As you'll see, one of the many advantages of AssemblyScript's tight syntax is that it maps _very closely_ to native WebAssembly instructions, producing incredibly small artifacts.

## Building Actors in AssemblyScript

Building an actor in AssemblyScript requires the use of the [AssemblyScript Actor API](https://github.com/wascc/wascc-actor-as/), which is still in its infancy. This API contains implementations of functions for all of the following waSCC capabilities:

* Blob/Object Store
* Event Streams
* HTTP Server
* Extras (random numbers, GUIDs, etc)
* Key-Value Store
* Message Broker
* Level-aware logging

If you're familiar with the existing Rust SDK, you may remember that the waSCC host runtime communicates with actors via the [message pack](https://msgpack.org/index.html) serialization format. It isn't as compact as protocol buffers, but the serialization/de-serialization latency is better, and, more importantly, message pack code is available across a larger variety of languages.

The brilliant folks working on [waPC](https://github.com/waPC) have not only written their own native AssemblyScript messagepack encoder/decoder, but they've also started working on a code generation DSL and framework (called **WIDL**) that allows developers to define their schema and (hopefully in the near future) generate code in any number of languages.

We'll be able to discuss at great length all of the details of WIDL code generation once that project matures and is less volatile. For now, let's take a look at how you create an actor in AssemblyScript.

### Creating an AssemblyScript Actor Project

The first thing we need to do is create an AssemblyScript project. The easiest way to do that today is to just copy a [package.json](https://github.com/wascc/examples/blob/master/kvcounter-as/package.json) file from an existing project. The 2 key dependencies you'll need in your project are:

```json
"dependencies": {
    "wascc-actor-as": "git+https://github.com/wascc/wascc-actor-as",
    "wapc-guest-as": "git+https://github.com/wapc/wapc-guest-as#v0.2.0",
},
```

We'll also be using `assemblyscript-json` in this project:

```json
"assemblyscript-json": "git+https://github.com/nearprotocol/assemblyscript-json"
```

Next, let's create the `index.ts` file in the `assembly` directory. The empty scaffold for this file looks as follows:

```js
import { handleCall, consoleLog, handleAbort } from "wapc-guest-as";

// TODO: your code eventually goes here


// Ceremony required for module entry points
export function _start(): void {
  // Set up message handlers
}

export function __guest_call(operation_size: usize, payload_size: usize): bool {
  return handleCall(operation_size, payload_size);
}

// Required Abort function
export function abort(
    message: string | null,
    fileName: string | null,
    lineNumber: u32,
    columnNumber: u32
  ): void {
    handleAbort(message, fileName, lineNumber, columnNumber);
  }
```

AssemblyScript doesn't have any of the "compiler magic" or custom macros that you get with Rust, so some of the internal WebAssembly stuff is laid bare before for you. Here, we need to give the actor's WebAssembly module an entry point (`__guest_call`), a `_start` function that sets up our message handlers, and an `abort` function that will deal with wasm aborts or "Traps". All of this plumbing exists in the Rust SDK, but it has been hidden from you. We're hoping that as we iterate on the AssemblyScript API, we can hide some of these details as well.

Now that we've got the basic scaffold up, let's define a message handler. For this post, we'll be re-creating the [Key-Value Counter](https://github.com/wascc/examples/tree/master/kvcounter) sample from the Rust SDK.

This actor handles an incoming HTTP request, atomically increments a key-value counter, and returns the new value as a JSON object. Let's start by declaring our message handler and modify the `_start()` function accordingly:

```js
export function _start(): void {
  Handlers.handleRequest(handleRequest);
}
```

This shows that we need to define a function called `handleRequest`, so let's do that now (put this anywhere you like in the file):

```js
function handleRequest(request: Request): Response {
  const kv = new KV("");
  const key = request.path.replaceAll("/", ":");
  const result = kv.atomicAdd(key, 1);  

  // TODO: convert result into JSON  

  return new ResponseBuilder()
    .withStatusCode(200)
    .withStatus("OK")
    .withBody(json.buffer)
    .build();
}
```

Here, we're using the `KV` class that comes with the API. There is a corresponding class for each of the first-party capabilities in waSCC, and it's easy for custom capability developers to provide their own. We strip the `/` off of the URL, use that as the key for the counter, and increment it.

Next, let's convert the result into JSON. Right now, AssemblyScript doesn't have a fancy way to automatically serialize objects into JSON, so we'll use the stack-style JSON builder from `assemblyscript-json` (put in place of the `TODO` from the previous code listing):

```js
let encoder = new JSONEncoder();

// Construct output JSON
encoder.pushObject("");
encoder.setInteger("count", result.value);  
encoder.popObject();

// Get serialized data
let json: Uint8Array = encoder.serialize();
```

## Building and Signing

We should now have a fully functioning actor. If we run `npm run build` (assuming we've run `npm install` first and you're using the `package.json` from the example), this will generate an actor WebAssembly module at `./build/kvcounter-as.wasm`

Finally, use `wascap` to sign the output module the same way you would any other actor module, and you're now ready to deploy and run your WebAssembly actor!

## Teeny Tiny Files

The last thing I want to mention before wrapping up this post is that AssemblyScript has such a close mapping to the native WebAssembly instruction set that _almost no_ excess code is generated. In fact, most of the actor binaries I've produced have been under **50KB**. The `kvcounter-as.wasm` file is only **19KB** when I compile it.

I think it's worth taking a moment to consider workload density in the cloud here. Rust-based actors can get as small as **1MB**. AssemblyScript actors can get as small as **20KB**. Imagine an entire cloud deployment of **actors** instead of microservices, and what kind of impact that might have on your overall cloud spend.
