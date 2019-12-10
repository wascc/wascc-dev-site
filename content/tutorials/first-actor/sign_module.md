---
title: Sign the Actor Module
linktitle: Sign Module
toc: true
type: docs
date: "2019-12-09T00:00:00Z"
draft: false
menu:
  first-actor:
#    parent: Example Topic
    weight: 3

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

In this step, we'll go through the process of signing an actor module. Signing modules is required because the **waSCC** host enforces that no actor can utilize capabilities to which it has not been granted access. 

Signing an actor module involves creating an _account key_ and a _module key_, and then signing the actor with those keys and a list of capabilities that actor will use.

## Install the Nkeys Tool
In order to generate your account and module keys, you'll need the [nkeys](https://github.com/encabulators/nkeys) tool installed. You can download the source and compile the binary yourself, or you can install the CLI with cargo:

```shell
$ cargo install nkeys --features "cli"
```

To verify that the _nkeys_ tool has installed properly, type `nk` in your shell and you should see output similar to the following:

```shell
$ nk
nk 0.0.7
Kevin Hoffman <alothien@gmail.com>
A tool for manipulating nkeys

USAGE:
    nk <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    gen     Generates a key pair
    help    Prints this message or the help of the given subcommand(s)
```

## Create an Account Key
An [account](/docs/security/accounts/) is an entity that issues a signed JWT (Json Web Token) which is then embedded in the module. In short, the account is a unit of trust. If you trust the account, then, assuming the signature is valid, you trust the module issued by that account.

To create a new account key, enter the following command:

```shell
$ nk gen account
Public Key: ABANVDKJYPTJZQTECHRJTTZONTJUWVPEG22T6DKLASAFYZ2DRGPQGJBV
Seed: SAAKPT6OUB3DUPKHGR4BXTTG6SZE5DEXIJ2LBOIF3CTLMOQ5F6FL2FH3NU

Remember that the seed is private, treat it as a secret.
```
Your output will vary since each key is unique. Copy the `seed` value (note the `SA` prefix) and store it in a file account `account.nk` as we'll need to use this file later. 

## Create a Module Key
An [actor module](/docs/security/modules/) is an entity of execution. It inherits its trust from the trust of of the _issuer_ (the account). Every actor we create gets a public key derived from its seed key.

To create a module key, enter the following command:

```shell
$ nk gen module
Public Key: MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR
Seed: SMAI2EIJNR3ODSRTGXXBKZWRRU6HHT3CETVBZ246AVWJYZ6GSUIR43A4UE

Remember that the seed is private, treat it as a secret.
```
Copy the `seed` value (note the `SM` prefix) into a file called `module.nk`. This will also be required for signing the module.

## Install the Wascap Tool
The [wascap](https://github.com/wascc/wascap) library (and its accompanying CLI) is used for embedding, extracting, and validating the signatures and capability attestations in actor modules. You'll need this installed before you can sign modules.

As with the other tooling, you can install `wascap` via `cargo install`:

```shell
$ cargo install wascap --features "cli"
```

## Sign your Module
Signing your module is just a matter of running `wascap` with the keys you generated earlier and specifying which capabilities this actor is allowed to use. In our case, the only capability the actor needs is the HTTP server (indicated with the `-s` flag):

```shell
$ wascap sign -s -a ./account.nk -m ./module.nk target/wasm32-unknown-unknown/debug/hellohttp.wasm hello_signed.wasm
Successfully signed hello_signed.wasm.
```

This will have produced a new file called `hello_signed.wasm`.

## Examine Module Signature
Before we continue on to the next step, let's take a look at what our signature looks like on the actor module:

```shell
$ wascap caps ./hello_signed.wasm 
╔════════════════════════════════════════════════════════════════════════════╗
║                          Secure WebAssembly Module                         ║
╠═══════════════╦════════════════════════════════════════════════════════════╣
║ Account       ║   ABANVDKJYPTJZQTECHRJTTZONTJUWVPEG22T6DKLASAFYZ2DRGPQGJBV ║
╠═══════════════╬════════════════════════════════════════════════════════════╣
║ Module        ║   MCYQSR5WIOABHZP6Z3SG67REVC2QDCYAHUVXHUSSLFWNO55OZ3O33MKR ║
╠═══════════════╬════════════════════════════════════════════════════════════╣
║ Expires       ║                                                      never ║
╠═══════════════╬════════════════════════════════════════════════════════════╣
║ Can Be Used   ║                                                immediately ║
╠═══════════════╩════════════════════════════════════════════════════════════╣
║                                Capabilities                                ║
╠════════════════════════════════════════════════════════════════════════════╣
║ HTTP Server                                                                ║
╠════════════════════════════════════════════════════════════════════════════╣
║                                    Tags                                    ║
╠════════════════════════════════════════════════════════════════════════════╣
║ None                                                                       ║
╚════════════════════════════════════════════════════════════════════════════╝
```

This gives us an idea of the kind of power we have in the embedded tokens. We didn't specify an expiration period or a time delay before the module can be used, so the token is valid right now. Note that the public keys for the account and module correspond to the public keys produced by the `nk` tool. 

The most important piece is that the `HTTP Server` capability is the only one our new actor is allowed to use. This means that no matter what code we write or how many messages we attempt to send, the actor module _will not_ be allowed to utilize any other capability.

In the next part of the tutorial, you'll create a host runtime to run this actor.