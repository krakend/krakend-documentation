---
lastmod: 2020-03-27
date: 2017-01-21
linktitle: Server requirements
menu:
  community_current:
    parent: "110 Deployment and Go-Live"
title: KrakenD servers requirements
weight: 10
---
When comparing KrakenD with other solutions, a key difference is that its **Total Cost of Ownership** is actually **lower**, and you can save a lot of money in infrastructure.

On the hardware side, KrakenD is **very light** and consumes very low resources. For instance, the **consumption pattern** of the *baseline* (we will see this definition below) is around `100-200MB` of RAM and can work on production with `0.5 vCPU`. This *baseline* can process thousands of requests per second.

## Hardware specifications
So, what are the server specifications (or container limits) I need to set for KrakenD?

*It depends*. For starters, there is **no minimum size** needed to operate KrakenD, and there is no certain type of hardware that you need to have. You can use anything from bare-metal to virtualized environments and Docker containers. In fact, you can run KrakenD in the **small** virtual servers with very good results.

{{< note title="Determining the server size" type="note" >}}
Your final **server size** or **container limit** depends on many factors, like your choosen configuration and enabled components, the performance of your backends, the timeout settings, the network, the concurrency, and other factors.

The proper right way to determine this size is **load testing the gateway**.

Whatever is the server size you choose, the recommended approach is having **more small machines rather than one or two big ones**.

{{< /note >}}

Keep an eye on **network usage**, as it is the first limit that KrakenD hits on public-cloud instances with limited throughput. Your KrakenD machines can be at a 10% of CPU/memory usage and still you can reach the maximum amount of traffic your provider allows you to use.

### Guide to server sizing
Let's put some generalistic examples so you can quickly get an approximation in the following table. The recommended size is **per KrakenD node**, and assuming you have **high throughput**. And try to scale horizontally rather than vertically. You can use these numbers to start with sizing, albeit they are not written in stone:

| vCPUs      | Memory | Gateway features and purpose         |
| ---------- | -------| ------------------------------------ |
| `0.5CPU` | `512MB`|  **Baseline** gateway (see definition below) with high throughput. No token validation, nor memory/cpu intensive operations and transformations.
| `1CPU` | `512MB`|  **Baseline** gateway plus light cryptography algorithms in token validation, and moderate logging.
| `2vCPU`   | `2GB`  |  **All purpose gateway**  |
| `4vCPU`   | `4GB`  | **Massive traffic** and **several memory/cpu intensive operations, complex cryptography**. |
| `+4vCPU`   | `+4GB`  | Numbers above this will be when you use KrakenD to cache content, make use of high-memory or CPU functionalities massively, or your environment pushes the gateway to consume more resources. **Machines like this might indicate that you have a hidden problem**. |

{{< note title="What is the baseline in the table?" type="question" >}}
The baseline is a gateway with the following specifications enabled:
- Routing
- Encoding
- Filters and basic manipulations
- Circuit breaker
- High throughput
- No logging, neither token validation
{{< /note >}}

### Top 10 Memory and CPU consumption patterns
When extending the baseline, there are a few usual suspects responsible for memory or CPU consumption **when the througput is high**. Computationally complex components put the gateway under more pressure, and a downsized machine will show sympthoms of suffering on CPU/Memory. This is the **TOP 10** usages that make the gateway work more:

1. [High, missconfigured, or absent timeouts](/docs/enterprise/throttling/timeouts/) (*memory*): Let's say your backend is delaying its response while KrakenD keeps adding more and more new connections. In this scenario KrakenD will increase its memory consumption to hold those waiting for response users. Set low timeouts when possible (e.g: `3s`) and what is more important, that **KrakenD timeouts are larger than backend timeouts**. You don't want KrakenD cutting connections while your backend keeps working on something the end-user will never see.
2. [Lua scripts](/docs/enterprise/endpoints/lua/#supported-lua-types-cheatsheet) (*memory and CPU*) : They run in their own virtual context and need to be compiled in every execution. They also open the door to any code you can write that can lead to blocking operations. It's up to you if you add an implementation *O(1)* or *O(N)*. If used in endpoints with a high hit-rate, consider moving the logic to a Go plugin: it won't change any complexity you might have added but it will perform several times faster.
3. [Backend caching](/docs/backends/caching/) (*memory*): KrakenD does not limit the cache size you can use in any way. Your backend is in charge (through the `Cache-Control` header) of deciding how much memory KrakenD will need, the cardinality, and for how long!. Make sure you do not let the KrakenD cache go over its memory limit to see failures in its execution.
4. [Per-user rate limiting](/docs/endpoints/rate-limit/) (*memory*): User rate limiting requires KrakenD to store one unique counter per connected user, and per endpoint. The number of users and endpoints you have impacts on the total memory needed to store these limits. Pay attention to memory if you expect thousands of users or endpoints concurrently, and the length of the key you want to tokenize, as you will need to reserve memory to those counters.
5. [Token validation](/docs/authorization/jwt-validation/) or [signing](/docs/authorization/jwt-signing/) (*CPU*): Signing and validating JWT are in the top 5 operations with bigger computational cost. Try to choose least expensive algorithms (like `HS256`) over very expensive algorithms like the `RS256`. A special emphasys on high SHA algorithms like `RS512` as they can make you dimension your CPU to **x2.5** or more compared to `RS256`. If extra security is a must, consider using `ES256` instead of `RS512` to avoid a big impact. The more cryptographic complexity you add, the more CPU you are going to need.
6. [Flatmap](/docs/backends/flatmap/#flatmap-configuration) (*CPU*): If you have large responses of endless arrays and you need to traverse this large number of elements one by one to apply transformations, then prepare for extra time processing those manipulations.
7. [Over-inspection in Telemetry](/docs/telemetry/overview/) (*CPU and memory*): Enabling several systems of Telemetry at the same time makes KrakenD work harder to report the metrics to all systems. Ore when you have just one, but you have a lot of throughput and your metrics have a high cardinality, with all instrumentation layers enabled and you choose not to do sampling.
8. [Inadequate logging level](/docs/logging/extended-logging/) (*CPU and memory*): Having a `DEBUG` logging level in production versus not having logging at all has a huge difference in throughput (between x2 and x5 vs the *baseline*). Choose a level that provides you sufficient information to make decisions on production, and avoid having KrakenD writing every single detail of the interaction, and dedicate those resources to your end-users.
9. The [Bloomfilter](docs/authorization/revoking-tokens/) (*memory*): Although it is very effective and space efficient, yet it can consume a lot of memory depending on its configuration, as you decide the maximum dataset there.
10. **Regular expressions** in general (*CPU*): For instance when you need to validate requests with the [Common Expression Language](/docs/enterprise/endpoints/common-expression-language-cel), or when you need to [detect bots](/docs/throttling/botdetector/).

All the components above are resource-efficient but they offer full flexibility. Depending on the configuration you choose they can lead to an increased consumption of resources when compared with the baseline.