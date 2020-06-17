+++
title = "Actor Bindings"
linktitle = "Actor Bindings"
date = 2020-04-02

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 45

[menu.docs]
  parent = "concepts"
  weight = 20
+++

An _actor binding_ is a di-directional association between an actor and a capability provider. In most cases, an actor will only ever have a single relationship with a given capability provider, however, there can be many.

For example, assume that your actor communicates solely using the `wascc:messaging` capability. It might want a separate connection for a _control plane_ and a connection for the main _application logic_. In this case, there would be two actor bindings:

* A binding to `wascc:messaging` named `controlplane`
* A binding to `wascc:messaging` named `application`

While these bindings are named, they are _logical names_ and do not create any form of tight coupling between actor and provider. You can still swap out the provider of any given binding _at runtime_ without impacting the actor. You're free to use two different providers (e.g. NATS and RabbitMQ) to satisfy the bindings, or you can just use different connections to the same provider.

Because the use of just a single binding will be the most common scenario, if you supply a `None` (in the case of the Rust API) for a binding, then the binding named `default` will be used.

Even if you do not supply any configuration information at the time of binding, a capability provider _cannot communicate_ with an actor until a binding has been established (though the reverse can potentially be done if the provider requires no configuration information).

<small>Actor bindings were added in the <strong>0.6</strong> release</small>