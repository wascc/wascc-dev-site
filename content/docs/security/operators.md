+++
title = "Operators"
linktitle = "Operators"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 65

[menu.docs]
  parent = "security"
  weight = 3
+++

An **operator** is an entity that sits at the root of a _chain of provenance_ or an _authorization tree_. Operators grant legitimacy to [accounts](./accounts) by signing their JWTs, and in turn accounts grant legitimacy to [actors](./modules) by signing their JWTs.

The JWTs of an authorization tree are not tightly coupled, and it is entirely up to the ecosystem how to use them. Provenance is verified by traversing the authorization tree by following an _issuer chain_ until reaching the root, which terminates at an operator.
