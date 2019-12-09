---
title: Stack Trader
summary: An online, realtime multiplayer game written in WebAssembly
tags:
- space
- gaming
date: "2019-10-27T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Stack Trader Game UI
  focal_point: Smart

links:
- icon: twitter
  icon_pack: fab
  name: Follow waSCC
  url: https://twitter.com/wascc_runtime
url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
#slides: example
---
When we create new frameworks or libraries, we often exercise these libraries with the canonical _"hello world"_ example. This example is typically the simplest possible thing that will exercise the functionality. I have a number of problems with the use of _hello world_ samples to demonstrate libraries, especially complex ones involving distributed systems.

When we started working on [waSCC](https://github.com/wascc), it was an experiment to see if we could run low-privilege, secure, isolated, portable WebAssembly modules in the cloud as services and functions. Very early on in the process, we were able to build and run a microservice using **waSCC**.

We needed to build something that would create a real-world scenario for developers; something that had complex requirements and would illustrate the core tenets of waSCC. We wanted to make sure that we could:

* Reduce time to market (reduce the _"mean time from napkin to production"_)
* Build secure, tamper-proof WebAssembly modules
* Maintain the ability to dynamically and horizontally scale that we already know we can accomplish with docker and container scheduling platforms like k8s 
* Deploy everything on Kubernetes
* Handle real-world load
* Flip the "boilerplate ratio", by spending **90%** of our time on features and only **10%** of our time on boilerplate/non-functional requirements.

To do this, we actually built two things:

* [dECS Cloud](https://github.com/wascc/decs-cloud) - A distributed Entity-Component-System engine
* [Stack Trader](https://github.com/wascc/stack-trader) - A real-time, online multiplayer game written using **dECS Cloud**

The project was a resounding success. We were able to build **dECS Cloud** in such a way that if we needed more system managers, we could simply spin up more host processes for the system manager WebAssembly module. We could shard traffic easily because we chose [NATS](https://nats.io) as our underlying broker technology. 

In the span of just about 4 weeks, we were able to create a back-end game engine _and_ an example game, including the game's user interface. This was possible in such a short period of time because we had successfully flipped the _boilerplate ratio_, and had spent nearly all of our time working on features.

Other than one issue we had where we'd discovered a bug in the _key-value store provider_, all of our debugging time was devoted to the architecture and design of the game and game engine itself. **waSCC** _just worked_. We never had any issues with the WebAssembly runtime. Security worked as we expected it to, and we were able to detect and verify the right WebAssembly modules in our production cluster that we used for the KubeCon demonstration.

Put another way, we spent **99%** of our time _debugging features_, and not messing around in the weeds of non-functional requirements, dependency hell, and frameworks or copy/paste mistakes. It wasn't just a _refreshing_ experience, it was actually _enjoyable_.

We learned a lot about how consumers might want to work with a cloud-native host runtime for WebAssembly modules--which was the entire point of the exercise. The learning we took from this experiment was rolled directly into what is now the current version of **waSCC** and it fed the future feature roadmap.

Take a look at the video of the game running that we demonstrated during KubeCon 2019 in San Diego, CA:

<iframe width="560" height="315" src="https://www.youtube.com/embed/5k1BvuO6ZJQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

What's supporting the game being played in this video is a [Critical Stack](https://criticalstack.com) cluster with the following services deployed:

* System Manager (dECS Engine) - `wasm`
* Component Manager **x3 instances** (dECS Engine) - `wasm`
* Shard Manager (dECS Engine) - `wasm`
* Game Loop (dECS Engine) - `wasm`
* Dashboard (dECS Engine) - `React` (_no WebAssembly for the UI!_)
* Leaderboard System (Stack Trader) - `wasm`
* Merchant System (Stack Trader) - `wasm`
* Mining System (Stack Trader) - `wasm`
* Navigation System (Stack Trader) - `wasm`
* Physics System (Stack Trader) - `wasm`
* Radar System (Stack Trader) - `wasm`
* Game UI (Stack Trader) - `React` (_no WebAssembly for the UI!_)
* RESgate - `docker` (Bridge NATS messages to front-end Web Sockets)
* NATS - `docker` (Message Broker)
* Redis - `docker` (Key-Value Store)

When we stop and think about the sheer number of independently and horizontally scalable services we built in such a short period of time with just two developers working part-time-- where each service could be mocked and tested in isolation, each one had secure, verifiable provenance, and each one had almost _no boilerplate_, and each one consumed _**less than 2MB** of disk_--it is a pretty amazing thing!
