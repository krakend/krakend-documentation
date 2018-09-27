---
aliases:
- /features/other/
lastmod: 2018-09-27
date: 2016-09-30
toc: true
linktitle: Other
title: Other
weight: 100
menu:
  documentation:
    parent: features
---

# Load balancing

KrakenD has a round-robin based load balancing component for a proper load distribution against the registered backends. This component is totally transparent for the user.

# Service Discovery

KrakenD integrates several Service Discovery strategies. Each one can be associated with a set of endpoints of the service configuration at the same time.

## `static`

This strategy uses directly the hosts declared at the backend without any further manipulation.

## `dns`

This subscriber just gets the DNS SRV associated with the single service name passed in the backend config host array. The resultant set is refreshed every 30 seconds.

## `etcd`

Users can relay on the etcd client to watch for the instances related to the single service name passed in the backend config host array. Since the `etcd` keys can be watched individually, there is no polling frequency.

# Rate limit

The rate limits allow you to restrict the traffic to any component of the stack (the KrakenD itself and the actual API backend services) and mainly cover two different purposes:

- Avoid flooding your system with massive requests
- Establish a quota of usage for your API
- Create a simple QoS strategy for your API

These measures are complementary to the [Circuit Breaker](/docs/throttling/circuit-breaker).

Check out the [Rate limit](/docs/throttling/rate-limit) section for more info about this component.

# Circuit breaker

To keep KrakenD responsive and resilient, we added a **Circuit Breaker** middleware on several points of the processing pipe. Thanks to this component, when KrakenD demands more throughput than your actual API stack is able to deliver properly, the **Circuit Breaker** mechanism will detect the failures and prevent stressing your servers by not sending requests that are likely to fail. The **Circuit Breaker** is also useful for dealing with network and other communication problems, by preventing too many requests to fail due to timeouts, etc.

The **Circuit Breaker** is a very simple **state machine** in the middle of the request and response that monitors all
the failures of your backend and when they reach a configured threshold the circuit breaker will prevent sending more
traffic to the suffering backend.

The Circuit Breaker is a protection measure for your stack and avoids cascading failures. It is **always enabled** (and is transparent to you).

Check out the [Circuit Breaker](/docs/throttling/circuit-breaker) section for more info about this component.
