---
lastmod: 2024-11-27
old_version: true
date: 2016-07-01
linktitle: Circuit Breaker
title: Circuit Breaker
description: Implement the circuit breaker pattern in KrakenD API Gateway to enhance the resilience and stability of your API ecosystem
weight: 930
menu:
  community_v2.11:
    parent: "090 Traffic Management"
dark_header_image: true
images:
- /images/documentation/diagrams/circuit-breaker-overview.mmd.svg
- /images/documentation/circuit-breaker.png
- /images/documentation/circuit-breaker-states.png
meta:
  since: false
  source: https://github.com/krakend/krakend-circuitbreaker
  namespace:
  - qos/circuit-breaker
  scope:
  - backend
  log_prefix:
  - "[BACKEND: /foo][CB]"
---
The **Circuit Breaker** is a straightforward **state machine** in the middle of the request and response that monitors all your backend failures. In the image above you can see a simplified version of its behavior. When backends fail to succeed for a number of consecutive times, the circuit breaker will prevent sending more traffic to a failing backend alleviating its pressure under challenging conditions.

When KrakenD demands more throughput than your API stack can deliver properly, the Circuit Breaker mechanism will detect the failures and prevent stressing your servers by not sending requests that are likely to fail. It is also helpful for dealing with network and other communication problems by preventing too many requests from dying due to timeouts, etc.

It is important to remark that the number of maximum errors are **consecutive errors**, and **not the total** of errors in the period. This approach works better when your traffic is variable, as it's based on a **probabilistic pattern** and it's not affected by the volume of traffic you might have.

{{< note title="A must have" type="tip" >}}
The Circuit Breaker is an **automatic protection measure** for your API stack and **avoids cascade failures**, keeping your API responsive and resilient. It has a small consumption of resources. Try to implement it always. You might start with a big number of errors if you hesitate.
{{< /note >}}


## Circuit breaker configuration

The Circuit Breaker is available in the namespace `qos/circuit-breaker` inside the `extra_config` key of every `backend`. The following configuration is an example of how to add circuit breaker capabilities to a backend:
```json
{
    "endpoints": [
    {
        "endpoint": "/myendpoint",
        "method": "GET",
        "backend": [
        {
            "host": [
                "http://127.0.0.1:8080"
            ],
            "url_pattern": "/mybackend-endpoint",
            "extra_config": {
                "qos/circuit-breaker": {
                    "interval": 60,
                    "timeout": 10,
                    "max_errors": 1,
                    "name": "cb-myendpoint-1",
                    "log_status_change": true
                }
            }
        }
        ]
    }
    ]
}
```

The attributes available for the configuration are:

{{< schema version="v2.11" data="qos/circuit-breaker.json" >}}

## How the Circuit Breaker works
It's easy to picture the state of the circuit breaker as an electrical component, where an open circuit means no flow of electricity between the ends, and a closed one normal flow:

<img title="Circuit Breaker states" src="/images/documentation/circuit-breaker.png" class="dark-version-available">

The Circuit Breaker starts with the `CLOSED` state, meaning the electricity can flow to the backends as they are considered healthy (*innocent until proven guilty*).

Then the component watches the state of the connections with your backend(s), with a tolerance to **consecutive failures** (`max_errors`) during a time interval (`interval`). it stops all the interaction with the backend for the next N seconds (the `timeout`). We call this state `OPEN`.

After waiting for this time window, the state changes to `HALF-OPEN` and allows **a single connection** to pass and **test the system** again. At this stage:
- If the test connection fails, the state returns to "open" and the circuit breaker will wait N seconds again to test it again.
- If it succeeds, it will return to the "closed "state,  and the system is considered healthy.

This is the way the states change:

![circuit-breaker-transitions.mmd diagram](/images/documentation/diagrams/circuit-breaker-transitions.mmd.svg)

- `CLOSED`: In the initial state, the system is healthy and sending connections to the backend.
- `OPEN`: When the consecutive number of errors from the backend (`max_errors`) is **exceeded**, the system changes to `OPEN`, and no further connections are sent to the backend. The system will stay in this state for N seconds ( where N = `timeout`).
- `HALF-OPEN`: After the timeout, it changes to this state and allows **one connection** to pass (the test). If the connection succeeds, the state changes to `CLOSED`, and the backend is considered to be healthy again. But if it fails, it switches back to `OPEN` for another timeout.

### Definition of error

When the circuit breaker counts the number of consecutive `max_errors`, an error could be anything that prevents having a successful connection with the service and completing the work.

{{< note title="`no-op` endpoints do not check HTTP status codes" type="warning" >}}
Because a `no-op` does not evaluate the response status code, the circuit breaker does not see the response status code of the backend and the errors are limited to the following list below.
{{< /note >}}


An error could be any of the following:

- Network or connectivity problems
- Security policies
- Timeouts
- Components in the list returning errors or having issues:
    - Proxy rate limit (`qos/ratelimit/proxy`)
    - Lua backend scripts (`modifier/lua-backend`)
    - CEL in the backend (`validation/cel`)
    - Lambda (`backend/lambda`)
    - AMQP or PubSub issues

#### For endpoints that DO NOT use `no-op`
In addition, only when you work with `json`, or **any other encoding different than `no-op`**, the gateway also takes into account the HTTP responses back from the backend and marks as errors:

- Status codes different than `200` or `201` (including client credentials)
- Decoding issues
- Martian issues
