---
aliases:
- /throttling/rate-limit/
- /docs/throttling/rate-limit/
lastmod: 2018-09-27
date: 2016-07-01
linktitle: Rate Limits
title: Router rate limiting
weight: 10
menu:
  documentation:
    parent: features
---



# Router: Rate limiting usage of KrakenD endpoints
The router rate limit is a **per endpoint** setting that allows you to set the number of **maximum requests per second** a KrakenD endpoint will accept. If no configuration is added explicitly KrakenD is not going to apply any limitation in the number of requests it can handle.

In order to specify a rate limit, you need to add the configuration in the desired endpoint. The router rate limit **does not accept a default value** to be used across all the endpoints. Nevertheless, when using the [KrakenDesigner](http://designer.krakend.io/) if you specify a default rate limit in the *Service* section it will copy the setting in every single endpoint for you when there is no setting.

At the router level, you can set the rate limit for endpoints based on:

1. Maximum request rate an endpoint accepts
2. Maximum request rate an endpoint accepts **per client**


## Maximum request rate per endpoint (`maxRate`)
Enable this option when you want to set the maximum requests an endpoint can handle within a window of 1 second.

When set, every KrakenD instance will keep in-memory an updated counter with the number of requests being processed per second in that endpoint.

The absence of `maxRate` in the configuration or `"maxRate": 0` mean NO LIMIT.

### What happens when the `maxRate` limit is reached?
If API users reach the established limit in the endpoint then KrakenD will start rejecting requests. Users will see an HTTP status code `503 Service Unavailable`.

## Maximum request rate per endpoint and client (`clientMaxRate`)
Similar to `maxRate` the `clientMaxRate` establishes a *user quota*. Limits the maximum requests per second the KrakenD service will process for a single endpoint and a given client. Every KrakenD instance will keep its counters of the requests per second **every single client** is doing.

There are two client identification strategies that can be chosen:

  - `"strategy": "ip"` When the restrictions are based on the client's IP, and every IP is considered a different user.
  - `"strategy": "header"` When the criteria for identifying a user comes from the value of a `key` inside the header. With this strategy the `key` must be also specified.
    - E.g, set `"key": "X-TOKEN"` to use the `X-TOKEN` header as the unique user identifier.

The absence of `clientMaxRate` in the configuration or `"clientMaxRate": 0` means NO LIMIT per user.

The value you use for `clientMaxRate` should be significantly lower than you would use for `maxRate`, as this limit will be multiplied by the number of active clients at that moment.

Example:

    clientMaxRate : 10

If 200 different customers (with different IPs) hit the KrakenD, they will be allowed to generate a total traffic of

    200 ips x 10 req/s = 2000req/s

 {{% note title="Perf note" %}}
 From a **performance point of view**, the `clientMaxRate` drops the performance as every incoming client needs to be monitored. Use it wisely.
 {{% /note %}}

### What happens when the `clientMaxRate` limit is reached?
If an API user (IP or header strategies) reaches the established limit in the endpoint then KrakenD will start rejecting requests. The user will see an HTTP status code `429 Too Many Requests`.

## Router rate limit example, `clientMaxRate`, and `maxRate` together.
The following example demonstrates a configuration with several endpoints, each one setting different limits:

- A `/happy-hour` endpoint with unlimited usage as it sets `0`.
- A `/happy-hour-2` endpoint is also unlimited as it sets no rate configuration.
- A `/limited-endpoint` is capped at 50 reqs/s AND their users can make up to 5 reqs/s (where a user is a different IP)
- A `/user-limited-endpoint` is not limited globally but every user (identified with `X-Auth-Token` can make up to 10 reqs/sec)

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


