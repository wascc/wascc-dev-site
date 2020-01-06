+++
title = "Accounts"
linktitle = "Accounts"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 3

[menu.docs]
  parent = "security"
  weight = 3
+++

An **account** is a _certifying authority_ for a module. Put another way, accounts are the _issuers_ of module tokens. There is no enforced limit on the number of module tokens an account can issue. How many accounts you choose to use will depend on the type of security infrastructure you have and the kind of verification you do in your waSCC runtime hosts.

As mentioned earlier, waSCC _does not_ have access to any secrets in order to perform verification. So, when you write code to add a security hook to your waSCC runtime host, you will have access to the `issuer` field of the module's token, which will correspond to the _public key_ of an account.

Public keys of accounts (and any other entity generated through ed25519 [nkeys](https://github.com/encabulators/nkeys)) do not change so long as the private key does not change. This makes them a source of reliable and unique identity.
