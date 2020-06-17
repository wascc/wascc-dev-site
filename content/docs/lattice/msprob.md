+++
title = "The Microservices Problem"
linktitle = "The Microservices Problem"
date = 2020-06-10

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 96

[menu.docs]
  parent = "lattice"
  weight = 2
+++

The problem with microservices is not that they are too small, not that they aren't small enough, and not even that the term itself is now watered-down industry jargon. No, the problem with microservices is that _they are tightly coupled to the processes in which they are hosted_.

If we're diligent adherents of the latest microservices software trends, then we've taken a look at our domain model (or our existing legacy monolith) and we've taken a first good stab at dividing things up along the lines of [bounded contexts](https://martinfowler.com/bliki/BoundedContext.html). Depending on the size of our application, we could have hundreds of microservices that need to be coded, tested, deployed, and maintained.

## The Medium is not the Message

Imagine you're told that you need to create a service that rates insurance policies. Our natural, knee-jerk instinct reaction to this problem might be to say things like, _"I'll create a RESTful endpoint that does ...."_ or _"Let's create a lambda that reacts to an HTTP trigger from ..."_. These statements are already on the wrong track, and not the way we should be thinking about creating _components_ to solve our problems.

_The medium through which input is delivered to our services should not be a logical part of our service_. I know this may sound heretical to some, but we're opinionated here and firmly believe that when you build a service, you shouldn't have to build it with HTTP, JSON, TCP, UDP, AMQP, MQTT, WebSockets, gRPC, Kafka, Lambda Events, or any other specific medium in mind. Your service has inputs, and it needs to perform a task which results in outputs and/or side-effects. _That_ is where our focus should be, and _that_ is where we should spend most of our time.

Traditional microservice development tightly couples the medium and the message, often before we've even written the first line of code. This is not good enough for the future of development for the cloud and the edge.

## Processes are Inflexible

_An operating system binary is not an agile deployment artifact_. Operating System processes are by definition tightly coupled to the host operating system and the host system architecture. Even though we want to build simple, testable business logic, we need to deal with the reality that, at least in today's world, we're usually responsible for building and deploying this process. We might use systems like [Kubernetes](https://kubernetes.io/) to schedule, monitor, and keep our processes up and running, but developers are still responsible for this thing as a unit of deployment; or worse, as part of a larger unit of deployment like a [docker](https://www.docker.com/) container.

_Process-binding prevents our units of business logic from being composable_. With the technology most of us are using today, we can't take two services that used to be running on their own on different nodes and recombine them into a single process without at least rebuilding and re-deploying services, and in many cases also re-designing or re-architecting the innards of the services involved.

## Utilization is Inefficient

Imagine you've got a large application consisting of hundreds of microservices deployed in an enterprise. Each of these services carries the code to start whatever listeners are required for whichever medium we've chosen to deliver messages to the service. It contains the compiled library code to talk directly to all kinds of dependencies. In a tiny, small corner of that process lies our business logic, the _real reason_ we wrote the code; the source of value of that service.

All of this overhead is wasteful, and [FaaS](https://en.wikipedia.org/wiki/Function_as_a_service) platforms recognize this and try and strip that away, reducing your business logic down to a "reactive component", or a distributed functional call, but usually with trade-offs that tightly couple other aspects of your business logic to the FaaS platform.

Further, if we're deploying our services in a docker-based cluster, then chances are we're running at a pretty low utilization rate on each of the nodes in that cluster, even if we've been smart about our designs and implementations.

## Services are Environment-Bound

We like to tell ourselves that our code will run anywhere, but that's usually something we tell ourselves to feel better (or a truth we hide behind automation). I can test my code on my laptop, and I can deploy my code to production by shipping it with a manifest or binding it to configuration and runtime secrets, but does that code _really_ run anywhere? In truth, our services can usually only run in places that have a direct, behind-the-firewall connection to our databases, and that have a bunch of ceremony and overhead involving security and compliance. There are URLs our services can't call, service meshes and proxies altering our traffic, SSL certificates we need installed, and other myriad tiny details that must be in place in each environment to make our services work.

Microservices we build using today's standards can't just be lifted out of one cloud and placed in another and expected to work[^1]. We can't take a service that's running in AWS right now and _move_ it to an edge-deployed location and expect it to continue working with no downtime.

Our services are environment (or at least environment _profile_) bound today, but they don't have to be.

[^1]: _We can certainly layer complexity upon complexity and code to make this possible, but it's not an intrinsic quality of microservices_.

## We Deserve Better

Developers are great at adapting. We adapt to difficult situations and we can quickly get used to nearly any source of friction. But _we shouldn't have to_. Coding should be a _joy_, developing and deploying to the cloud should be _fun_, _secure_, and _reliable_. We think we can do better by building a **[Cloud Native Hypervisor](../hypervisor)**.
