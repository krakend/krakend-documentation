---
lastmod: 2018-09-29
aliases: ["/docs/throttling/"]
date: 2016-07-01
linktitle: Throttling overview
title: Traffic management overview
notoc: true
weight: -10
menu:
  community_current:
    parent: "070 Traffic Management"
---
KrakenD offers several ways to protect the usage of your infrastructure that might act at very different levels.

The most significant type of traffic management feature is the **rate limit**, which allows you to throttle the traffic of end-users or the traffic of KrakenD against your backend services. The *rate limits* mainly cover the following purposes:

- Avoid stressing or flooding your backend services with massive requests (proxy rate limit)
- Establish a quota of usage for your exposed API (router rate limit)
- Create a simple QoS strategy for your API

Traffic Management covers:

- [Circuit Breaker](/docs/backends/circuit-breaker/): An automatic protection measure for your stack and avoids cascade failures.
- **Rate-limiting**:
  - [Service Rate Limiting (stateless) {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/service-settings/service-rate-limit/): Sets the maximum throughput users can have to a KrakenD instance.
  - [Redis-based global rate limit (stateful) {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/throttling/global-rate-limit/): Sets the maximum throughput users can have on a KrakenD cluster, backed on Redis.
  - [Endpoint Rate Limiting](/docs/endpoints/rate-limit/): Sets the maximum throughput all connected users can have against specific endpoints (stateless).
  - [Client Rate Limiting](/docs/endpoints/rate-limit/): Sets the maximum throughput each end-user has to specific endpoints (stateless).
  - [Proxy Rate Limiting](/docs/backends/rate-limit/): Sets the maximum throughput KrakenD can have between an endpoint and your backend services (stateless).
- [Spike Arrest](/docs/throttling/spike-arrest/): Ensures a minimum time between different requests goes by
- [Service Discovery](/docs/backends/service-discovery/): To detect and locate services automatically on your enterprise network.
- [Bot detection](/docs/throttling/botdetector/): Reject bots carrying out scraping, content theft, and other forms of spam.

You can combine parts or all the traffic management features.