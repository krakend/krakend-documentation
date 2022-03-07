---
lastmod: 2018-09-29
date: 2016-07-01
linktitle: Throttling overview
title: Traffic management overview
weight: -10
menu:
  community_current:
    parent: "070 Traffic Management"
---
KrakenD offers several ways to protect the usage of your infrastructure that might act at very different levels.

The most significant type of traffic management feature is the **rate limit** that allows you to throttle the traffic of end-users or the traffic of KrakenD against your backend services. The *rate limits* mainly cover the following purposes:

- Avoid stressing or flooding your backend services with massive requests (proxy rate limit)
- Establish a quota of usage for your exposed API (router rate limit)
- Create a simple QoS strategy for your API

Traffic Management covers:

- [Circuit Breaker](/docs/backends/circuit-breaker/): An automatic protection measure for your stack and avoids cascade failures.
- **Rate-limiting**:
  - [Endpoint Rate Limiting](/docs/endpoints/rate-limit/): Sets a maximum throughput to all users hitting KrakenD endpoints.
  - [Client Rate Limiting](/docs/endpoints/rate-limit/): Sets individual throughput to end-users hitting KrakenD endpoints.
  - [Proxy Rate Limiting](/docs/backends/rate-limit/): Sets a maximum throughput between KrakenD and your backend services
- [Spike Arrest](/docs/throttling/spike-arrest/): Ensures a minimum time between different requests goes by
- [Service Discovery](/docs/backends/service-discovery/): To detect and locate services automatically on your enterprise network.
- [Bot detection](/docs/throttling/botdetector/): Reject bots carrying out scraping, content theft, and other forms of spam.
