+++
title = "Gantry Overview"
linktitle = "Gantry Overview"
date = 2020-05-21

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 1

[menu.docs]
  parent = "gantry"
  weight = 1
+++

**[Gantry](https://github.com/wascc/gantry)** is a registry for waSCC actor modules and an authorization hierarchy. It is responsible for maintaining the tokens for operators, accounts, and actors as well as the raw bytes for each revision of every actor. Gantry makes the guarantee that if an entity exists within the registry, then its token and chain of provenance has been vetted. If you're querying or downloading a module from Gantry, the client can be confident that the files are legitimate and have not been tampered with.

Gantry _does not_ communicate over HTTP, but opts instead for the more flexible NATS transport.

Gantry enables waSCC runtime hosts to be flexibly deployed to any location as determined by a scheduler (e.g. Kubernetes) and then request the raw bytes for an actor's WebAssembly module on demand after the runtime has started. It can also be used to facilitate zero-downtime (with no transient resource scaling!) updates.

The Gantry server requires a root operator JWT at the time of launch, providing the root node of the authentication tree. This runtime injection of the root node enables scenarios where different operator hierarchies are used in production, staging, development, etc. 

Gantry can be thought of as _"like docker hub, but for waSCC actors,"_ though that is an over-simplification of the crucial role it performs and the power it can provide for a waSCC runtime environment.
