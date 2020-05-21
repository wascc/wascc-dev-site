+++
title = "Gantry Operator Hierarchies"
linktitle = "Operator Hierarchies"
date = 2020-05-21

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 2

[menu.docs]
  parent = "gantry"
  weight = 2
+++

## Operator Hierarchies

Gantry supports the notion of _secure multi-tenancy_ by default. When the Gantry server starts up, it requires that an _operator token_ be passed to it on the command line (which, of course, can come from an environment variable, a container-mounted volume, etc). Remember that this JWT is verifiable offline, and _contains no secrets_.

Each Gantry server _catalog_ (managed by the [catalog](https://github.com/wascc/gantry/tree/master/catalog) actor) can store and validate multiple operators. These operators, in turn, verify multiple accounts, as illustrated in the following diagram:

[![](https://mermaid.ink/img/eyJjb2RlIjoiZ3JhcGggVEJcbiAgICBBKFtSb290IE9wZXJhdG9yXSktLT58S2V5IEF8QltPcGVyYXRvciAxXVxuICAgIEEtLT58S2V5IEJ8Q1tPcGVyYXRvciAyXVxuICAgIEEtLT58S2V5IEJ8RFtPcGVyYXRvciAzXVxuICAgIEItLT58S2V5IEF8QUMxKEFjY291bnQgQSlcbiAgICBCLS0-fEtleSBCfEFDNChBY2NvdW50IEUpXG4gICAgQy0tPnxLZXkgQ3xBQzIoQWNjb3VudCBCKVxuICAgIEQtLT58S2V5IER8QUMzKEFjY291bnQgQylcbiAgICBBQzQtLT58S2V5IEF8TTFbQWN0b3IgQV1cbiAgICBBQzQtLT58S2V5IEJ8TTJbQWN0b3IgQl1cbiAgICBBQzQtLT58S2V5IEN8TTNbQWN0b3IgQ10iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoiZ3JhcGggVEJcbiAgICBBKFtSb290IE9wZXJhdG9yXSktLT58S2V5IEF8QltPcGVyYXRvciAxXVxuICAgIEEtLT58S2V5IEJ8Q1tPcGVyYXRvciAyXVxuICAgIEEtLT58S2V5IEJ8RFtPcGVyYXRvciAzXVxuICAgIEItLT58S2V5IEF8QUMxKEFjY291bnQgQSlcbiAgICBCLS0-fEtleSBCfEFDNChBY2NvdW50IEUpXG4gICAgQy0tPnxLZXkgQ3xBQzIoQWNjb3VudCBCKVxuICAgIEQtLT58S2V5IER8QUMzKEFjY291bnQgQylcbiAgICBBQzQtLT58S2V5IEF8TTFbQWN0b3IgQV1cbiAgICBBQzQtLT58S2V5IEJ8TTJbQWN0b3IgQl1cbiAgICBBQzQtLT58S2V5IEN8TTNbQWN0b3IgQ10iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)

There are a couple of very good reasons for the `gantry-server` application requiring the runtime injection of a _root operator token_. The root operator token contains the list of that operator's _public_ signing keys. This is amounts to a "whitelist" of the operators that can exist in the catalog (anything issued by any of the signing keys). You can think of this as requiring a _second factor_ that, when when activated, validates an entire catalog of tokens. _New operators cannot be inserted into the database unless they were signed by one of the keys referenced in this token_.

Without this additional step, if an intruder were able to gain write access to the database, they could fabricate their own root operator, mint new operators issued by that one, and continue this until they crafted malicious (but valid) actor tokens. With this approach to the operator hierarchy, an intruder needs to have _both_ write access to the database _and_ at least one private signing key from the root operator to mint fake tokens.
