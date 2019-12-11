---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "WebAssembly Host Runtimes - More than just a J2EE Evolution"
subtitle: "Are we repeating the mistakes of the past?"
summary: "The comparison between wasm host runtimes and J2EE is inevitable. Is this just a new face on an old technology?"
authors: []
tags: []
categories: []
date: 2019-12-11T08:05:06-05:00
lastmod: 2019-12-11T08:05:06-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
Collectively, our industry has a long and difficult history of re-inventing things as something that may still not solve the original problem. Those who adhere to the _not invented here_ (NIH) philosophy are constantly re-building things as their own. All too often, we end up re-inventing the wheel as a square simply so that we can claim that this new thing is _our wheel_.

Are we doing that with WebAssembly outside the browser? If you take only a cursory glance at some of the features of wasm host runtimes in the cloud, it might indeed seem like we're just re-inventing [J2EE](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition).

**J2EE** is a _specification_. It is an agreement between an application and a supporting runtime that the runtime will satisfy some set of requirements. As the word _"enterprise"_ is actually in the spec name, J2EE runtimes were designed for the enterprise. They sought to facilitate things like the creation of web services and distributed computing, and took care of things like scalability, distribution, and concurrency. J2EE is, in my opinion, very much an inspiration for modern-day workload scheduling platforms like Kubernetes.

J2EE had _reference runtimes_--products made by different vendors (some open source, some not) that purported to conform to the J2EE specification. The WebAssembly community has something similar to this. We have a specification called **[WASI](https://wasi.dev)** that is a contract between a host runtime and a wasm file guaranteeing that certain low-level, POSIX-like system calls will be available. J2EE runtimes guaranteed that various enterprise-level services would be available.

## Sandboxes and Sidecars
The first and likely most important difference between J2EE and host runtimes is that a host runtime is a true, isolated sandbox whereas J2EE is actually more akin to today's concept of a [sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar) that provides a suite of supporting, opt-in services to an application. 

A J2EE runtime _does not constrain_[^1] the applications whereas with WebAssembly, it's physically impossible for the wasm code to do anything beyond the sandbox. Even the system-level WASI calls are _imports_, and as such must go through the host runtime and are only permitted at the behest of that host. I've seen dozens of applications running in production that made their own outbound HTTP (or message broker, database, etc) calls without going through the J2EE container, essentially circumventing the sandbox.

## Security
Security was one of the biggest motivating factors in our work to build **waSCC** and it's definitely one that weighs heavily on the minds of anyone who has had to roll out newly built docker images to production because of discovered vulnerabilities, or those who have had rogue workloads scheduled into their clusters to do all kinds of nefarious things.

J2EE reference runtimes from Oracle and IBM and open source implementations all had security. In general, you had the ability to tag Java application bundles with a set of roles, granting it certain privileges or restrictions within the runtime. However, it was pretty easy to tamper with application bundles and poorly configured host runtimes were a popular attack vector.

This is precisely what runtimes like waSCC aim to fix. We can embed cryptographic signatures inside a `wasm` file that can securely prove the unique identity of the module, its capabilities (in J2EE terms, that would be the _roles_ the application is allowed), and other important metadata that not only travels with the module, but cannot be separated from it without invalidating it.

## Java Bytecode vs WebAssembly Bytecode
Again, on the surface, these things look pretty similar. Both `.wasm` files and `.jar` files contain _[bytecode](https://techterms.com/definition/bytecode)_, which is basically a low-level format designed to be _interpreted_ (or interpreted and then compiled into a non-portable native format).

Java's portability was one of the biggest promises of the language when it came out, and, with some exceptions, it delivered on that promise. The difference again lies within the host runtime implementations. While the J2EE specification was pretty clear on which features were supported, every vendor had their own special implementation with subtle differences and, more importantly, _extra features_. Enterprise developers wrote code tightly coupled to those features and suddenly you had portable _Java_ code that was _not portable across J2EE runtimes_, negating many of the benefits of both the language and the runtime.

**WASI** is a low-level specification, and **waPC** is a higher-level spec upon which **waSCC** is built. Portability of bytecode here depends entirely on the implementation of the host runtimes. While I think we're all looking back on the J2EE days as a way to learn from our mistakes, only time will tell if we end up fragmenting the wasm community with incompatible runtimes. My hope with **waSCC** is that other developers will build their own runtimes on top of it, and so will be able to provide specializations without tightly coupling the `wasm` modules built by application developers.

Another feature we get with wasm is the ability to send compiled code to a live process. This, again, is something that we could (sort-of) do with J2EE, and again the comparison is one of evolution. With a runtime like waSCC we can ship in-place, zero-downtime upgrades without the need to bring up transient resources (like we need to do with Kubernetes rolling upgrades). You could push an update to a J2EE application while it was live, but it wasn't fast, it didn't scale well, and it rarely ever qualified as "zero downtime".

## Performance, Size, and Workload Density
waSCC (and I suspect other similar runtimes) is smaller, faster, and more efficient than J2EE. It is designed to support modern applications, services, or functions running dynamically anywhere they are needed.

Our experiments show that even some of the larger microservices built on waSCC are only **2MB** compiled `wasm` files. Today's enterprises are rightfully concerned with workload density--the number of workloads they can squeeze onto a single host to maximize utilization and reduce cost overhead.

J2EE was massive and bloated, as were the applications they hosted. They were designed to be monoliths. With just a few MB of RAM overhead from the waSCC runtime, you can literaly stuff _hundreds_ of services into the same footprint we can run a single standalone application written in modern Java with Spring Boot or a handful of Go applications.

## Conclusion
Are out-of-browser WebAssembly host runtimes just another J2EE? **No**. However, J2EE and runtimes like waSCC endeavor to solve the same kinds of problems. We need a way to  reduce boilerplate, speed time to production, remove friction from every step of the development process, and we need to be able to build applications that can run anywhere we deploy them, regardless of the host characteristics.

Most of us hated the world of J2EE. It was a high-friction development process with an error-prone, brittle deployment process and an aged configuration mechanism steeped in the world of pets for computing instead of cattle. That said, Fortune 500 companies relied on J2EE runtimes as the bedrock of their server-side operations and did so for years. For all its faults, J2EE solved real problems.

So perhaps WebAssembly host runtimes that deliver enterprise capabilities are just an evolution of J2EE--but I think we've learned from our mistakes and if we do this right, we can build the cloud native version of what J2EE _should have been_. The workloads are smaller, faster, arguably more portable, easier to deploy, more secure, and benefit from a stronger sandbox.

I think we have the potential to deliver solutions to the new problems we face as we move to the cloud, to the edge, and to embedded devices in a way that was better than J2EE. However, whether we succeed or are remembered as another failed re-invention is entirely up to us and the community, the tooling, and the technology we deliver. It's our game to lose.

[^1]: This varied by runtime. Some products (especially the commercial ones) had some ways to constrain applications, but a huge problem with this was that each environment from workstation to production could have different sandbox settings.