+++
title = "Separation of Keys"
linktitle = "Key Separation"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 80

[menu.docs]
  parent = "security"
  weight = 6
+++

As described throughout the documentation, decentralized security within the **waSCC** ecosystem relies on the idea that _secrets are never required to verify or authorize entities_. This is a subtle, but powerful concept. It means that the full, production system can be up and running without ever having to hold onto a secret. By extension, this means that an entire production environment could be compromised and _not a single secret would be lost_.

_We would rather adopt a security model that gives nothing of value to successful intruders rather than hopes that no one ever successfully intrudes._

There are some best practices that we have enabled in some cases and enforced in others around the separation of keys.

## Operator Practice (Mandated by Gantry)

An operator sits at the root of a chain of authorization. This means that it is the _root authority_. Being the root of the tree means that, by definition, operator tokens are _self-signed_ or _self-issued_. The issuer and subject of an operator are always the same. If the _identity key_ (the key used to _issue_ the operator's token) of the operator is compromised, then the operator itself will have to be invalidated, causing a cascading effect that invalidates all accounts issued by that operator. The theoretical blast radius of such a compromise could be the entire operator tree.

As such, a common best practice of segregating the identity key from multiple signing keys is actually mandated by the Gantry product. To prevent a cascading failure, the operator self-issues with an identity and then only ever issues accounts with _signing keys_. It can use as many signing keys as is desired, but the requirement is that the identity key is separate from the signing keys. In fact, the identity key of an operator is often kept in offline storage and never allowed to venture into a cloud or network environment at all.

By following this practice, if the identity key of an operator is compromised, we can invalidate/blacklist the compromised key and re-issue a new operator token with a new signature _while still maintaining the secure chain of authorization for all previously issued accounts_. This is possible because the signing key(s) for the accounts remain valid. More importantly, _an introduder cannot mint imposter tokens for accounts with a stolen operator identity key_ because this practice considers accounts issued by an operator identity key to be invalid.

Because this practice is so essential to the safety of an operator hierarchy, Gantry will reject the provenance chain of accounts that were signed by an operator's identity key.

Further, if an organization institutes a practice of regular _key rotation_, then separating the identity keys from signing keys allows the signing keys to be rotated while the public keys of the entities within a tree remain the same (and can thus be relied upon not to change for use as fixed global identifiers good enough to use in application configuration).

The wascap tool will allow you to specify the identity key and additional signing keys for an operator.

**WARNING:** _the wascap tool will not stop you from creating a self-issued operator with no additional signing keys, which will thus be incompatible with Gantry._

## Account Practice

While we _strongly recommend_ the use of _signing keys_ for accounts, this is optional when using the Gantry product. Because the loss of an account doesn't destroy an entire authorization tree, some organizations consider it an acceptable risk, especially for accounts only responsible for a small number of actors. 

The wascap tooling will allow you to specify any number of additional signing keys. The _public keys_ of those signing keys will appear in a field called `valid_signers` in the `wascap` metadata field of the token. This means that the public JWT for an account can be used to verify any actor by comparing the `iss` field of an actor with any of the keys in the `valid_signers` list.

## Actor (Module) Practice

Actors, at least in the current waSCC ecosystem, do not need signing keys, since they are not the issuing authority for any entity. Actors are the "leaves" in a tree, and thus they only have identity keys. The loss of an identity key for an actor is actually a relatively harmless exposure because the only thing an intruder could do would be to _self-issue_ a token for an actor, and self-issued actor tokens are already considered invalid by the system.

## Summary

_The more signing keys you have, the smaller the blast radius caused by key compromise_. It is up to you and your organization to choose how many signing keys you want and to balance the management burden of keys against the risk mitigation of using multiple keys.
