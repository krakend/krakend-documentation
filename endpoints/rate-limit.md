---
aliases:
- /throttling/rate-limit/
- /docs/throttling/rate-limit/
- /docs/features/rate-limit/
lastmod: 2018-10-30
date: 2016-07-01
linktitle: Rate Limits
title: Endpoint rate limiting
weight: 10
source: https://github.com/devopsfaith/krakend-ratelimit
menu:
  documentation:
    parent: endpoints
---

Limiting endpoints is the responsibility of the **router rate** and allows you to set the number of **maximum requests per second** a KrakenD endpoint will accept. By default, there is no limitation on the number of requests an endpoint can handle.

To specify a rate limit, you need to add the configuration in the desired endpoint.

At the router level, you can set the rate limit for endpoints based on:

1. Maximum number of requests an endpoint accepts in a second (`maxRate`)
2. Maximum number of requests an endpoint accepts **per client** (`clientMaxRate`)


## Endpoint rate limit (`maxRate`)
Enable this option when you want to set the maximum requests an endpoint can handle within a window of 1 second.

When set, every KrakenD instance keeps in-memory an updated counter with the number of requests processed per second in that endpoint.

The absence of `maxRate` in the configuration or `"maxRate": 0` is the equivalent to no limitation.

### What if the `maxRate` limit is reached?
If API users reach the established limit in the endpoint, then KrakenD starts rejecting requests. Users see an HTTP status code `503 Service Unavailable`.

## Endpoint rate limit per client (`clientMaxRate`)
The rate per client works similarly to `maxRate`, but the `clientMaxRate` establishes a *user quota*.

Instead of counting all the connections to the endpoint, the `clientMaxRate` keeps a counter for every client and endpoint. Keep in mind that every KrakenD instance keeps its counters in memory for **every single client**.

**Example**:

    clientMaxRate : 10

200 different customers (with different IPs) hitting the limited KrakenD endpoint are allowed to generate total traffic of:

    200 IPs x 10 req/s = 2000req/s


{{< note title="A note on performance" >}}
Limiting endpoints per user makes KrakenD keep in memory counters for the two dimensions: *endpoints x clients*.

The `clientMaxRate` drops the performance as every incoming client needs tracking. Even counters are small in data it's easy to end up with several millions of counters.  Make sure to do your math every time.
{{< /note >}}

The absence of `clientMaxRate` in the configuration or `"clientMaxRate": 0` is the equivalent to no limitation.

### Client identification strategies
Two **client identification strategies** are available:

  - `"strategy": "ip"` When the restrictions apply to the client's IP, and every IP is considered to be a different user. Optionally a `key` can be used to extract the IP from a custom header:
    - E.g, set `"key": "X-Original-Forwarded-For"` to extract the IP from a header containing a list of space-separated IPs (will take the first one). 
  - `"strategy": "header"` When the criteria for identifying a user comes from the value of a `key` inside the header. With this strategy, the `key` must also be present.
    - E.g., set `"key": "X-TOKEN"` to use the `X-TOKEN` header as the unique user identifier.

### What if the `clientMaxRate` limit is reached?
If an API user (IP or header strategies) reaches the established limit in the endpoint, KrakenD starts to reject that specific client's requests. The user sees an HTTP status code `429 Too Many Requests`.

## Example of  `clientMaxRate` and `maxRate` together.
The following example demonstrates a configuration with several endpoints, each one setting different limits:

- A `/happy-hour` endpoint with unlimited usage as it sets `0`.
- A `/happy-hour-2` endpoint is also unlimited as it sets no rate configuration.
- A `/limited-endpoint` is capped at 50 reqs/s, AND their users can make up to 5 reqs/s (where a user is a different IP)
- A `/user-limited-endpoint` is not limited globally, but every user (identified with `X-Auth-Token` can make up to 10 reqs/sec)

Example

    {
        "version": 2,
        "endpoints": [
          {
              "endpoint": "/happy-hour",
              "extra_config": {
                  "github.com/devopsfaith/krakend-ratelimit/juju/router": {
                      "maxRate": 0,
                      "clientMaxRate": 0
                  }
              }
              ...
          },
          {
              "endpoint": "/happy-hour-2",
              ...
          },
          {
              "endpoint": "/limited-endpoint",
              "extra_config": {
                "github.com/devopsfaith/krakend-ratelimit/juju/router": {
                    "maxRate": 50,
                    "clientMaxRate": 5,
                    "strategy": "ip"
                  }
              },
              ...
          },
          {
              "endpoint": "/user-limited-endpoint",
              "extra_config": {
                "github.com/devopsfaith/krakend-ratelimit/juju/router": {
                    "clientMaxRate": 10,
                    "strategy": "header",
                    "key": "X-Auth-Token"
                  }
              },
              ...
          }
