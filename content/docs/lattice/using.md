+++
title = "Using waSCC Lattices"
linktitle = "Using Lattices"
date = 2020-06-10

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 99

[menu.docs]
  parent = "lattice"
  weight = 5
+++

As mentioned in the previous section, lattices are self-forming and should _just work_. Working with waSCC hosts and lattices should be just as easy as working with isolated waSCC hosts, except that the host now makes its actors and capability providers available to other lattice members.

## Building a Lattice-Enabled Host

Creating a lattice-enabled host is as simple as enabling the `lattice` feature. If you're building your own binary, you can modify your `wascc-host` dependency in your `Cargo.toml` file as follows:

```
wascc-host = { version = "0.9.0", features = ["gantry", "manifest", "lattice"]}
```

or if you're building the general-purpose `wascc-host` binary, you can simply compile it with the following command line:

```
$ cargo build --features "bin manifest lattice"
```

When you start a waSCC host with lattice mode enabled, you'll see the following line of `stdout` logging (assuming you have `INFO` level enabled):

```
[2020-06-09T20:38:19Z INFO ] Initialized Message Bus (lattice)
```

By default, the waSCC host doesn't have this feature enabled and you'll see the following startup output:

```
[2020-06-09T20:38:19Z INFO ] Initialized Message Bus (internal)
```

## Introducing the Leafcar

A **_leafcar_** is a portmanteau of `leaf node` and `sidecar`. Leaf node refers to running a NATS server as a leaf node, while [sidecar](https://www.oreilly.com/library/view/designing-distributed-systems/9781491983638/ch02.html) refers to running that NATS server bound to the loopback adapter (`127.0.0.1`) where it is not listening on any external IP address. The default mode of accessing the lattice message bus is through simple authentication against the NATS server running on the local loopback (there is usually one per `node` (either physical or virtual)), but you can supply configuration or environment variables that can do things like point to a fixed address for NATS, authenticate with signed tokens, etc.

The immediate benefit of the **_leafcar_** is that all traffic is optimized for local delivery. Unless there is a subscriber to an invocation on the other side of the leaf node, no traffic will leave. This means that you have full control over how much or little of your traffic flows across to different portions of your infrastructure. You can choose your logical traffic segmentation regardless of the underlying physical network topology.

## Forming a Lattice

Forming a lattice is the easy part--it _just works_. If you're running on your laptop and you've got a NATS server bound to loopback (you can get the small server binary or run it as a docker image), then the first waSCC host process you start will become a "lattice of one." The second one you start joins the lattice, and so on. There's no `yaml`, no 300-page manual on low-level networking required, and no complicated or error-prone supervisor-subordinate negotiations. The lattice as perceived by the hosts and the components within them is a flat topology, regardless of how many intervening leaf nodes and cluster servers there are.

Function invocations between actors and capability providers will automatically stay on the local node unless one of the parties is remote, where the invocation will traverse the shortest path to reach the remote host, even if it's in another cloud or on a Raspberry Pi.

## Examples

You can use all existing waSCC samples in lattice mode, because the existence of a lattice is _completely transparent_ to all actors and capability providers in the ecosystem. You don't need to write special code to accommodate this, or even recompile any of your build artifacts. To see this in action, first start a NATS server with no configuration parameters on the loopback adapter:

```
❯ ./nats-server -a 127.0.0.1
```

If you don't have (or want to build) the binary, you can run it from a docker image.

Next, we're going to start 3 waSCC hosts, and none of these hosts will be able to satisfy a request on their own. Let's take a look at the manifest files for these hosts:

**Hosts 2 and 3**:

```
# Loads a host that starts off with nothing but an unbound HTTP server provider
# NOTE that there are no actors loaded
---
    actors: []   
    capabilities:        
        - path: ./examples/.assets/libwascc_httpsrv.so
    bindings: []
```

**Host 1**:

```
# This host is to contain nothing but the echo server actor.
# NOTE that the HTTP server provider is NOT included in this host.
# Ensure that you launch host2.yaml and host3.yaml before launching this one.
---
    actors:
        - ./examples/.assets/echo.wasm
    capabilities: []
    bindings:
        - actor: "MB4OLDIC3TCZ4Q4TGGOVAZC43VXFE2JQVRAXQMQFXUCREOOFEKOKZTY2"
          capability: "wascc:http_server"
          values:
            PORT: "8081"
        - actor: "MB4OLDIC3TCZ4Q4TGGOVAZC43VXFE2JQVRAXQMQFXUCREOOFEKOKZTY2"
          capability: "wascc:http_server"
          values:
            PORT: "8082"
```

From the root directory of the `wascc-host` project, make sure you've build the binary with lattice enabled and run the following commands, in this order. Make sure you execute each of these from a different terminal tab or window--you want 3 processes running at the same time.

```
$ ./target/debug/wascc-host --manifest examples/lattice/host3.yaml
$ ./target/debug/wascc-host --manifest examples/lattice/host2.yaml
$ ./target/debug/wascc-host --manifest examples/lattice/host1.yaml
```

Now you can execute the "echo server" example the same way you did without lattice mode. The HTTP servers configured will respond to your requests on posts `8081` and `8082`. When you make an HTTP request via `curl localhost:808x/foo/bar`, you'll see the request handled by either host2 or host3, and you'll see the actor be invoked in host1. Because lattice communication among copies of the same entity (e.g. the `wascc:http_server` provider) is random, you may see one host process remain idle while the other starts 2 servers, or you could see the servers split evenly among the hosts.

## Conclusion

_That's it_. You should be able to simply "flip the switch" to enable lattice, and all of your workloads and capability providers can now be deployed in an elastically scalable distributed environment, no matter what infrastructure you're using underneath.
