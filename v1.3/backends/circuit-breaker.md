---
lastmod: 2021-05-02
old_version: true
date: 2016-07-01
linktitle: Circuit Breaker
title: The Circuit Breaker
weight: 20
menu:
  community_v1.3:
    parent: "050 Backends Configuration"
images:
- /images/documentation/circuit-breaker.png
- /images/documentation/circuit-breaker-states.png
notoc: true
meta:
  since: false
  source: https://github.com/krakendio/krakend-circuitbreaker
  namespace:
  - github.com/krakendio/krakend-circuitbreaker/gobreaker
  scope:
  - backend
---

To keep KrakenD responsive and resilient, we added a **Circuit Breaker** middleware on several processing pipe points. Thanks to this component, when KrakenD demands more throughput than your actual API stack can deliver properly, the Circuit Breaker mechanism will detect the failures and prevent stressing your servers by not sending requests that are likely to fail. It is also useful for dealing with network and other communication problems by preventing too many requests to fail due to timeouts, etc.

The **Circuit Breaker** is a straightforward **state machine** in the middle of the request and response that monitors all your backend failures. When they reach a configured threshold, the circuit breaker will prevent sending more traffic to the suffering backend.

The Circuit Breaker is a protection measure for your stack and avoids cascading failures.

## Circuit breaker configuration

The Circuit Breaker is available by default in KrakenD thanks to the [circuit breaker middleware](https://github.com/krakendio/krakend-circuitbreaker). As with all additional middlewares, you need to set its values in its own namespace `github.com/krakendio/krakend-circuitbreaker/gobreaker` inside the `extra_config` key.

The following configuration is an example of how to add circuit breaker capabilities to a backend:
{{< highlight json >}}
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
                "github.com/krakendio/krakend-circuitbreaker/gobreaker": {
                    "interval": 60,
                    "timeout": 10,
                    "maxErrors": 1,
                    "name": "cb-myendpoint-1",
                    "logStatusChange": true
                }
            }
        }
        ]
    }
    ]
}
{{< /highlight >}}

The attributes available for the configuration are:

- `interval`: (*integer*) Time window where the errors count, in seconds.
- `timeout`: (*integer*) For how long the circuit breaker will wait before testing again that the backend is healthy.
- `maxErrors`: (*integer*) The consecutive number of errors within the `interval` window to consider the backend unhealthy.
- `name`: (*string*) A friendly name to identify this circuit breaker's activity in the logs.
- `logStatusChange`: (*boolean*)  Whether to log the changes of state of this circuit breaker or not.

## How it works

The Circuit Breaker retains the state of the connections to your backend (s) over a series of requests
and when it sees more than the configured number of **consecutive failures** (`maxErrors`) in a given time interval (`interval`)
it stops all the interaction with the backend for the next N seconds (the `timeout`). After waiting for this time window, the system will allow a single connection to trial the system again: if it fails, it will wait N seconds more, and if it succeeds, it will return to the normal state, and the system is considered healthy.

The circuit breaker works with three different internal states, and the easiest way to imagine it is like in an electrical circuit:

| Circuit Breaker |
|-----------|
| ![Krakend logo](/images/documentation/circuit-breaker.png) |

- `CLOSED`: This is the normal state. When the circuit is closed, the electricity flows uninterrupted, and the connection to the backend is allowed.
- `OPEN`: No connection to the backend is allowed when the circuit is open.
- `HALF-OPEN`: When the system has seen repeated problems, only the necessary connection to test the backend is permitted.

And this is the way the states change:

| Circuit Breaker transitions |
|-----|
| ![Krakend logo](/images/documentation/circuit-breaker-states.png) |

- `CLOSED`: In the initial state, the system is healthy and sending connections to the backend.
- `OPEN`: When a consecutive number of supported errors from the backend (`maxErrors`) is exceeded, the system changes to `OPEN`, and no further connections are sent to the backend. The system will stay in `OPEN` state for N seconds ( the `timeout`).
- `HALF-OPEN`: After the timeout, it changes to this state and allows one connection to pass. If the connection succeeds, the state changes to `CLOSED`, and the backend is considered to be healthy again. But if it fails, it switches back to `OPEN` for another timeout.
