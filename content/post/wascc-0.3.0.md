---
title: "waSCC 0.3.0 Release Notes"
subtitle: "Actor-to-Actor Comms, Blob Store, and Actor Configuration"
summary: "This release adds several notable new features to the waSCC host runtime, including actor-to-actor communications, per-actor configuration, and the creation of a blob store capability standard and an initial file system capability provider."
authors: ["kevin"]
tags: []
categories: ["release-notes"]
date: 2020-02-18T07:13:41-05:00
draft: false
---

There are a number of exciting new features in this release. We found a few bugs, but the most notable one was an _uroboros_ bug where a thread would stall waiting for its tail to complete. You could reproduce this by publishing a message within an actor's message handler. This bug was introduced during the migration from the old codebase to the new multi-actor waSCC host.

## Actor to Actor Communications

At its heart, waSCC is a _dispatcher_ and a _multiplexer_. It is responsible for managing the flow of traffic between capability providers and actors, and doing so in a way that insulates the single-threaded developer code inside an actor from the complexities of multi-threaded code beyond the boundaries of the actor.

When working on fixing the aforementioned bug, I noticed that it would only take a few minor adjustments to allow one actor to make a call to another. To do this with your own actors, you'll need to do the following:

* Ensure that the actor providing the service and the actor consuming it both know the _public key_ of the actor. This will change if you change signing keys, so your development pipeline should be able to handle this.
* Sign the consuming actor with a "custom capability" that is identical to the providing actor's public key. This gives one actor the ability to call another. So instead of granting an actor a capability identifier like `wascap:messaging`, you'd grant it the public key of an actor module, e.g. `MCIXJVXAXKDX7UFYDFW2737SHVIRNZILS3ULODGEQOVCTWQ7HSGOHUY7`.
 It's important to note here that this system is _secure by default_, and no actor can communicate with any other actor (or capability) without being explicitly granted that privilege via `wascap` JWTs.
 
 There are some interesting use cases and subtleties here:
 
 * A providing actor can be granted more capability permissions than the consumer, which means if you grant one actor the ability to talk to another, you could potentially escalate privilege. In situations like this, guard your APIs well and make sure the actors only handle their limited set of messages.
 * Changing a signing key can potentially prevent you from talking to the target actor because it also changes the public key, so automating this process is crucial. Allowing an actor to be _configured_ with the identity of the actor it consumes (see next section) can be extremely powerful and help automate key changes in a development pipeline.


## Actor Configuration

Actors can now be configured. This doesn't mean that I condone _stateful actors_. It means that sometimes actors need configuration values that are provided by the outside world and we need a way to facilitate this. For example, if you're using _actor-to-actor_ comms and you want to configure the public key (subject) of the actor you're talking to, you can now inject that value at runtime via configuration.

Configuration should _not_ be used to manage state, nor should you use it to inject secrets or  credentials into an actor. 

To do this, just use `host::configure` as always, but both the _source_ and _target_ of the configuration will be the actor's public key. Take a look at the following example that informs an actor of a root operator JWT that might be used for something like provenance verification inside an actor:

```rust
  host::configure(
        "MCIXJVXAXKDX7UFYDFW2737SHVIRNZILS3ULODGEQOVCTWQ7HSGOHUY7",
        "MCIXJVXAXKDX7UFYDFW2737SHVIRNZILS3ULODGEQOVCTWQ7HSGOHUY7",
        operator_config(
            &operator.subject,
            &valid_signers,
        ),
    )?;
```

## Blob Store and the File System Capability Provider

One capability that I've thought waSCC needed to support "out of the box" has always been a blob store. Blob stores, or _object stores_, are really just places you can stuff arbitrarily large (within limits) binary objects or files in the cloud. Amazon, Azure, and Google all have blob storage capabilities and access to object stores has lately become a mandatory part of the default set of capabilities for cloud native applications.

With this release, we have defined the `wascap:blobstore` capability protocol (which can be found in `wascc-codec v0.3.0+`). One thing that's always bugged me about cloud blob stores is how difficult they are to play with and test when building locally. Ultimately we end up having to use some kind of simulator to pretend that our hard drive is an S3 bucket, etc.

With this release we've also created the first version of the _File System Provider_, a `wascap:blobstore` provider that exposes the blobstore API over top of an arbitrary file system directory. You configure the provider by giving it the location of its "root", and then it treats sub-directories below that as _containers_ and files within those sub-directories as _blobs_.

Because of the loosely coupled nature of capability providers, your actor should be able to switch between a file system, Azure, or Amazon blob store without recompilation.

### Constrained Environments and Blob Stores

We want actors to be _small_, _fast_, and _stateless_. This seems to conflict with some of the things we do with blob stores, like reading and writing potentially giant files in synchronous operations. If we exposed a single function like `download_blob` to the actor SDK, then we could grind that actor to a halt. While the waSCC host will make sure no messages are lost, a backlog of work will continue to queue up behind that large file transfer, and clients with short timeouts will give up.

What we need to do is allow actors to define _business logic_ (features) that operate on these blobs without grinding the WebAssembly host runtime to a halt in the process. The trick to this is _streaming_. If, instead of dealing with the raw blob bytes as a single array of `u8` values, we deal with them as a stream of "file chunks", then the waSCC host/VM can come up for air in between handling chunks and allow messages from other capability providers like a message broker to interleave with the file transfer messages.

WebAssembly modules start with a single `64KB` memory _page_, and it is up to the host runtime to grant that module more pages. If an actor had to address an entire blob from start to finish in a single memory space, we might also cripple the WebAssembly host runtime through over-allocation or, if the runtime denies the _grow_ request, we'll stop the actor's handler and it'll never finish.

This is why the actor SDK has the following functions related to blob store binaries that deal with the raw bytes as _streams of chunks_ rather than as monolithic memory-hogging vectors.

* `start_upload(blob, chunk_size, total_bytes)` - Notifies the capability provider that the actor is about to upload a blob, the chunk size you intend to use, and the file's total size. Some providers might use this information to allocate a cloud resource, while the file system provider can use it to verify that it can create a file of that size in the specified directory. The actor is then responsible for uploading individual chunks by calling `upload_chunk`.
* `start_download(blob, chunk_size)` - Notifies the capability provider that the actor is ready to download a file. If the request is successful, then the capability provider will start sending messages to the actor through the waSCC host with the operation `OP_RECEIVE_CHUNK`.

There should be enough metadata on the chunk structures to provide both sides enough context to manage these transfers, but we might have to iterate over the protocol a bit as more people use and develop blob store providers.

We will be updating the documentation and tutorials to include more information and samples on blob stores and the file system capability provider.

---
<br/>

I hope you enjoy the new features and I can't wait to see what everyone builds with waSCC!

