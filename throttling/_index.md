---
lastmod: 2023-11-01
aliases: ["/docs/throttling/"]
date: 2016-07-01
linktitle: Traffic Management Overview
title: "Traffic Management Overview"
description: Learn how KrakenD's traffic management features help optimize your API's performance, prevent abuse, and ensure a seamless user experience.
notoc: true
weight: 910
menu:
  community_current:
    parent: "090 Traffic Management"
---
Traffic management refers to the practice of **monitoring, controlling, and optimizing the flow of requests** to and from an API. It aims to prevent abuse by limiting the volume of requests from individual clients or groups, regulate the flow of traffic, ensure fair usage, and provide predictable API performance.

KrakenD offers several traffic management features, ranging from rate-limiting to advanced techniques like circuit breakers and bot detection. These features can be configured independently or combined for a holistic traffic management strategy.

You can **combine multiple traffic management features** to address complex use cases. You don't have to choose one or the other, but implement those that complete your needs

## Rate-Limiting
Rate-limiting controls the number of requests users or systems can send. KrakenD allows you to throttle both the traffic of end-users and the traffic of KrakenD against your services. The rate limits mainly cover the following purposes:

- Avoid stressing or flooding your backend services with massive requests (proxy rate limit)
- Establish a quota of usage for your exposed API (router rate limit)
- Create a simple QoS strategy for your API

Our approach to rate-limiting has many variants:

  - [Endpoint Rate Limiting](/docs/endpoints/rate-limit/): Sets the maximum throughput all connected users can have against specific endpoints (stateless).
  - [Client Rate Limiting](/docs/endpoints/rate-limit/): Sets the maximum throughput each end-user has to specific endpoints (stateless).
  - [Proxy Rate Limiting](/docs/backends/rate-limit/): Sets the maximum throughput KrakenD can have between an endpoint and your backend services (stateless).

Rate-Limiting features implement the [Spike Arrest](/docs/throttling/spike-arrest/), a mechanism triggered after exhausting the burst capacity of the rate-limit, ensuring that a minimum time interval occurs between consecutive requests, helping prevent sudden traffic spikes that could destabilize the system.


In addition, on the {{< badge >}}Enterprise{{< /badge >}} edition there is:

- [Service Rate Limiting (stateful, Redis)](/docs/enterprise/throttling/global-rate-limit/): Sets the maximum throughput users can have on a KrakenD cluster, backed on Redis. All machines centralize the counters in a database.
- [Service Rate Limiting (stateless)](/docs/enterprise/service-settings/service-rate-limit/): Sets the maximum throughput users can have to a KrakenD instance, but each machine counts its own traffic and no database.
- [Endpoint Rate Limiting (stateful)](/docs/enterprise/throttling/endpoint-redis-rate-limit/): Sets the maximum throughput users can have to specific endpoints, backed on Redis.
- [Tiered Rate Limiting (stateless)](/docs/enterprise/service-settings/tiered-rate-limit/#stateless-tiered-rate-limit): Same but the counters ara managed indepently by each machine and no database.
- [Tiered Rate Limiting (stateful, Redis)](/docs/enterprise/service-settings/tiered-rate-limit/#stateful-redis-backed-tiered-rate-limit): Sets the maximum throughtput users can send depending on their tier/plan at a cluster level, backed on Redis.

## Circuit Breaker
[Circuit Breakers](/docs/backends/circuit-breaker/) are automatic protection mechanisms that help prevent cascading failures in your system by temporarily halting requests to struggling backend services.

A simplified diagram would be:

![circuit-breaker-overview.mmd diagram](/images/documentation/diagrams/circuit-breaker-overview.mmd.svg)

The circuit breaker watches the state of the connections with your backend(s), with a tolerance to consecutive failures that you define. When the number of failures are reached, it stops all the interaction with the backend for a few seconds (timeout defined by you), and returns errors to clients. After the defined timeout, it tests the system again to see if it is already healthy or if it continues to fail. See the [Circuit Breaker](/docs/backends/circuit-breaker/) for more details.

## Bot Detection
[Bot detection](/docs/throttling/botdetector/) identifies and blocks malicious bots that scrape data, spam endpoints, or conduct other abusive behaviors. It allows you to set your own rules for bot detection, and they are based on regular expressions.

![bot detector](/images/krakend-botdetector.png)

## Geofencing
With [Geofencing {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/endpoints/geoip/), you can restrict API usage based on geographical locations, such as specific countries or cities. This feature, combined with [Security Policies](/docs/enterprise/security-policies/playbook/#user-is-from-a-specific-country), enhances regional access control.

When [GeoIP](/docs/enterprise/endpoints/geoip/) is enabled, the requests reaching to your backends have an additional enrichment of location metadata as well.

## IP Filtering
[IPFiltering {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/throttling/ipfilter/) allows you to block traffic from specific IP addresses, adding an extra layer of security.
