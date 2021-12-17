---
lastmod: 2020-03-27
date: 2017-01-21
linktitle: Server size
menu:
  community_current:
    parent: "110 Deployment and Go-Live"
title: KrakenD servers requirements
weight: 10
---
A key difference between using KrakenD over other solutions, is its **low Total Cost of Ownership**.

On the hardware side, KrakenD is **very light** and consumes low resources. Maybe even eco-friendly as a widely seen **consumption pattern** is around `100-200MB` of RAM and `0.5 vCPU` for installations that process a remarkable amount of traffic (thousands of requests per second).

The final number depends on many factors, including your choosen configuration, the surrounding environment and specially your backend behavior.

## Server size / container limits
The first question everyone has: **what is the server size I need for KrakenD?**

*It depends*. There is no minimum size needed to operate KrakenD, and in fact you can run it in the **smallest** virtual servers with good results. Despite the limits depend on your installation and the resource usage it will imply, in general terms, we can use this guide for an initial sizing idea of your nodes:

| CPUs      | Memory | Purpose                              |
| --------- | -------| ------------------------------------ |
| `0.5vCPU` | `512MB`|  Installations with a few thousands of requests per second without high-computation operations and transformations. |
| `2vCPU`   | `2GB`  |  Generic installations with **large traffic** and authorization |
| `4vCPU`   | `4GB`  | Configurations with massive traffic and several memory/cpu intensive operations. |

Numbers above this table will be when you use KrakenD to cache content, or make use og high-memory or CPU functionalities.

Keep an eye on **network usage**, as it is the first limit that KrakenD hits on public-cloud instances with limited throughput. Your KrakenD machines can be at a 10% of CPU/memory usage and still you can reach the maximum amount of traffic your provider allows you use.

### A few large servers, or several small servers?
Whatever is the server size you choose, the recommended approach is having more small machines rather than one or two big ones.

### Usual suspects of Memory and CPU consumption
KrakenD has a small memory and CPU **baseline** that will change according to the amount of work it has to do and the waits (queueing connections) for the backends to respond.

The baseline we are referring to is a gateway that does:

- Routing
- Basic encoding
- Basic filters and manipulations
- Circuit breaker
- No logging

When extending this baseline, there are a few usual suspects elements responsible for unusual memory or CPU consumption when the througput increases:

- [Lua scripts](/docs/enterprise/endpoints/lua/#supported-lua-types-cheatsheet) (affect memory and CPU) run in their own virtual context and need to be compiled in every execution. They also open the door to write blocking operations. If used in endpoints with a high hit-rate, consider moving the logic to a Go plugin.
- [Backend caching](/docs/backends/caching/) (memory): KrakenD does not limit the cache size you can use in any way. Your backend will decide through the `Cache-Control` headers how much memory KrakenD will need to store all the cache variety and for how long!. Make sure you do not let KrakenD cache go over its memory limit to see failures in its execution.
- [Per-user rate limiting](/docs/endpoints/rate-limit/) (memory): User rate limiting requires KrakenD to store one counter per connected user, and endpoint. The number of users and endpoints you have impacts on the total memory needed to store the limits.
- [Token validation](/docs/authorization/jwt-validation/) or [signing](/docs/authorization/jwt-signing/) (CPU). Signing and validating JWT are in the top 5 operations with bigger computational cost. Try to choose least expensive algorithms (like `HS256`) over very expensive algorithms like the `RSA256`. A special emphasys on high SHA algorithms like `RSA512` as they can make you dimension your CPU to **x2.5** or more compared to `RS256`.
- [Flatmap](/docs/backends/flatmap/#flatmap-configuration) manipulating endless arrays (CPU): If you have large responses of arrays that need to be manipulated with flatmap, prepare for extra time processing those manipulations.
- The [Bloomfilter](docs/authorization/revoking-tokens/) (memory) is very effective and space efficient but yet can consume a lot of memory.
- **Abscence or missconfigured timeouts** (memory and CPU): When your backend is not responding correctly, and KrakenD starts stacking a lot of users waiting your backend to respond, KrakenD will increase its memory consumption to hold those users. Make sure to set low timeouts whenever possible and that KrakenD timeouts are larger than backend timeouts.
- **Telemetry abuse** (CPU and memory): Enabling several systems of Telemetry at the same time makes KrakenD work harder to report the metrics to all systems.
- **Inadequate logging level**  (CPU and memory): Having a DEBUG logging level in production versus not having anything has a huge difference in throughput (between x2 and x5 vs the baseline). Choose the level that provides you useful information without having KrakenD writing every single thing it happened and dedicate the resources to your users.
- **Regular expressions** (CPU) to validate requests with [CEL](/docs/enterprise/endpoints/common-expression-language-cel) or [detect bots](/docs/throttling/botdetector/).
