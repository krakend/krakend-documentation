---
lastmod: 2023-11-01
old_version: true
date: 2016-07-01
linktitle: Throttling overview
title: "API Throttling Guide: Control Traffic & Prevent Abuse"
description: Implement efficient API throttling techniques to control traffic and prevent abuse. Follow our comprehensive guide to ensure fair usage of your API resources with KrakenD.
notoc: true
weight: 910
menu:
  community_v2.5:
    parent: "090 Traffic Management"
---
KrakenD offers several ways to protect the usage of your infrastructure that might act at very different levels.

The most significant type of traffic management feature is the **rate limit**, which allows you to throttle the traffic of end-users or the traffic of KrakenD against your backend services. The *rate limits* mainly cover the following purposes:

- Avoid stressing or flooding your backend services with massive requests (proxy rate limit)
- Establish a quota of usage for your exposed API (router rate limit)
- Create a simple QoS strategy for your API

In addition to rate-limiting, which is the most obvious functionality, when we talk about Traffic Management and API Throttling, KrakenD covers:

- [Circuit Breaker](/docs/v2.5/backends/circuit-breaker/): An automatic protection measure for your stack and avoids cascade failures.
- **Rate-limiting**, which has many variants:
  - [Service Rate Limiting (stateless) {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/service-settings/service-rate-limit/): Sets the maximum throughput users can have to a KrakenD instance.
  - [Redis-based global rate limit (stateful) {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/throttling/global-rate-limit/): Sets the maximum throughput users can have on a KrakenD cluster, backed on Redis.
  - [Endpoint Rate Limiting](/docs/v2.5/endpoints/rate-limit/): Sets the maximum throughput all connected users can have against specific endpoints (stateless).
  - [Client Rate Limiting](/docs/v2.5/endpoints/rate-limit/): Sets the maximum throughput each end-user has to specific endpoints (stateless).
  - [Proxy Rate Limiting](/docs/v2.5/backends/rate-limit/): Sets the maximum throughput KrakenD can have between an endpoint and your backend services (stateless).
- [Spike Arrest](/docs/v2.5/throttling/spike-arrest/): Ensures a minimum time between different requests goes by
- [Service Discovery](/docs/v2.5/backends/service-discovery/): To detect and locate services automatically on your enterprise network.
- [Bot detection](/docs/v2.5/throttling/botdetector/): Reject bots carrying out scraping, content theft, and other forms of spam.
- [Geofencing {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/endpoints/geoip/): With a combination of the GeoIP integration and [Security Policies](/docs/enterprise/security-policies/playbook/#user-is-from-a-specific-country) you can restrict usage based on country/city.
- [IPFiltering {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/throttling/ipfilter/) to block traffic from undesired IPs.

In KrakenD you can combine parts or all the traffic management features.