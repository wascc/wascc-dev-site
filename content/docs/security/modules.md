+++
title = "Modules"
linktitle = "Modules"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 5

[menu.docs]
  parent = "security"
  weight = 5
+++

A **module** is a _unit of deployment_ within a waSCC host runtime, also called an **actor**. This unit of deployment corresponds 1:1 to a `wasm` file (though it need not exist as a traditional file on disk). This file _must_ contain an embedded module token (JWT) in order to participate as an actor or capability provider in a waSCC host runtime.

Each module token has a `subject` which is the public key of the module. This should be used as the single source of truth for unique module identity within a waSCC ecosystem. Module tokens can be re-signed by accounts many times without changing the public key of the module. The only thing that can change the public key of a module is if the private _identity key_ needed to change.

The `issuer` of a module token is the _account_'s public key.

To see module signing in action, take a look at the [first actor](/tutorials/first-actor) tutorial.
