+++
title = "Security Overview"
linktitle = "Security Overview"
date = 2017-12-03

toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.
weight = 1

[menu.docs]
  parent = "security"
  weight = 1
+++

Securing microservices and cloud-native workloads is no small task, and is certainly something that should not be left as an afterhought. Integrating security into a cloud native development workflow from the beginning is difficult, and this difficulty is often a reason for delayed or incomplete implementations.

**waSCC** attempts to alleviate the burden of certain types of security concerns by enabling _security by default_ and allowing developers to practice _defense in depth_.

## Security in the Cloud

Deploying applications to the cloud means pushing them to an environment that might as well be a black box, and, in many cases, it is _someone else's_ black box. You might be using Amazon, Google, Microsoft, or any number of other vendors to host your application's supporting infrastructure.

In an environment like this, we can't afford to make assumptions about security. Our deployed applications must:

* Not be allowed to make unauthorized network connections
* Not be allowed to access host machine resources[^1]
* Not be allowed to run as a privileged user
* Not be allowed to consume too many resources (CPU, memory, etc)
* Not allow _impersonation_

In short, attackers shouldn't be able to pretend to be our applications in order to gain access to other parts of our system, nor should they be able to compromise a running application in order to force it to do something it wasn't intended to do.

In today's world, the restrictions placed on an application often exist only in the infrastructure, and so an application can have a different attack profile depending on the environment in which it is deployed. Having an application's security profile not tightly bound to the application itself is a security risk, and the waSCC documentation will illustrate how the waSCC runtime helps mitigate this risk.

## Security in a Containerized World

Containers have made all kinds of development models possible, and we can do things in the cloud with dynamically scaling compute that would've seemed impossible without innovations like **Docker** and the **Container Runtime Interface**.

However, even though they give us tremendous portability, containers are built with replaceable cake layers that are stacked to create the final running image. If any of these cake layers is compromised, it can immediately become a security risk or a source of catastrophic runtime failure.

In short, we can't achieve _defense in depth_ with containers unless we involve third party tools and do things like perform constant scans on all our layers. Today, even though we might not explicitly acknowledge it, we treat containers like a risk and a large potential attack vector.

This documentation will discuss how waSCC can eliminate an entire class of container-related risks and vulnerabilities.

[^1]: There are exceptions to every rule, though ideally we shouldn't violate this one
