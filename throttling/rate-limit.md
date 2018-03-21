---
aliases:
- /throttling/rate-limit/
lastmod: 2018-01-04
date: 2016-07-01
linktitle: Rate Limits
title: KrakenD - Rate limiting
weight: 10
menu:
  main:
    parent: Throttling and Limits
---

The rate limits allow you to restrict the traffic to any component of the stack (the KrakenD itself and the actual API backend services) and mainly cover two different purposes:

- Avoid flooding your system with massive requests
- Establish a quota of usage for your API
- Create a simple QoS strategy for you API

KrakenD provides rate limiting capabilities just by adding the desired configuration in the `krakend.json` file and this feature is complementary to the [Circuit Breaker](/docs/throttling/circuit-breaker), so they can exist together.

**The limits apply individually to each running instance of KrakenD**. For instance, if you limit an endpoint to 100 reqs/s in the `krakend.json` and you have 3 instances of KrakenD running your ecosystem is supporting 300 reqs/s.

The easiest way to play with rate limiting is through the [KrakenDesigner UI](http://designer.krakend.io/).

# Types of rate limits
There are two different clear layers where the rate limiting applies:

1. **Router layer**: Rate limit end-users hitting KrakenD endpoints
2. **Proxy layer** Rate limit KrakenD hitting your services

To deep dive in the internals see the [rate-limiting middleware](https://github.com/devopsfaith/krakend-ratelimit)

# Router: Rate limiting usage of KrakenD endpoints
The router rate limit is a **per endpoint** setting that allows you to set the number of **maximum requests per second** a KrakenD endpoint will accept. If no configuration is added explicitly KrakenD is not going to apply any limitation in the number of requests it can handle.

In order to specify a rate limit you need to add the configuration in the desired endpoint. The router rate limit **does not accept a default value** to be used across all the endpoints. Nevertheless, when using the [KrakenDesigner](http://designer.krakend.io/) if you specify a default rate limit in the *Service* section it will copy the setting in every single endpoint for you when there is no setting.

At the router level you can set the rate limit for endpoints based on:

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

There are two client identification strategies you can set (choose one):

  - `"strategy": "ip"` When you apply the restrictions based on the client IP. For you every IP is a different user.
  - `"strategy": "header"` When your criteria for identifying a user comes from the value of a `key` inside the header. You must specify both `strategy` and `key`.
    - E.g, set `"key": "X-TOKEN"` to use the `X-TOKEN` header as unique user identifier.


The absence of `clientMaxRate` in the configuration or `"clientMaxRate": 0` mean NO LIMIT per user.

The value you use for `clientMaxRate` should be significantly lower than you would use for `maxRate`, as this limit will be multiplied by the number of active clients in that moment.

Example:

    clientMaxRate : 10

If 200 different customers (with different IPs) hit the KrakenD, they will be allowed to generate a total traffic of

    200 ips x 10 req/s = 2000req/s

<div class="alert note">
    <h4><i class="icon fa fa-sticky-note-o"></i> Performance note!</h4>
    <p>From a <strong>performance point of view</strong> the <tt>clientMaxRate</tt> drops the performance as every incoming client needs to be monitored. Use it wisely.</p>
 </div>

### What happens when the `clientMaxRate` limit is reached?
If an API user (IP or header strategies) reaches the established limit in the endpoint then KrakenD will start rejecting requests. The user will see an HTTP status code `429 Too Many Requests`.

## Router rate limit example, `clientMaxRate` and `maxRate` together.
The following example demonstrates a configuration with several endpoints, each one setting different limits:

- A `/happy-hour` endpoint with unlimited in usage as it sets `0`.
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


# Proxy: Rate limit backends
No matter what is the activity users are generating at the router level, you might want to restrict the connections KrakenD makes to your backends. Configuration comes similarly than in the router case, but it's declared directly in the `backend` section instead of `endpoint` (or globally).

This parameter is defined at the `krakend.json` configuration file as follows:

    ...
    {
      "endpoint": "/products/{cat_id}",
      "backend": [
      {
          "host": [
              "http://some.api.com/"
          ],
          "url_pattern": "/catalog/category/{cat_id}.rss",
          "encoding": "rss",
          "extra_config": {
              "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
                  "maxRate": 1,
                  "capacity": 1
              }
          }
      }
    ...

There are two parameters you can set:

- `maxRate`: Maximum requests per second you want to accept in this backend.
- `capacity`: The capacity according to the [token bucket algorithm](https://en.wikipedia.org/wiki/Token_bucket) with a `bucket capacity == tokens added per second` so KrakenD is able to allow some bursting on the request rates. Recommended value is capacity==maxRate



# Comparison of `maxRate` vs `clientMaxRate`
The `maxRate` (available both in router and proxy layers) is an absolute number where you have the exact control on how much traffic you are allowing to hit the backend or endpoint. In an eventual DDoS the `maxRate` can help in a way since it won't accept more traffic than allowed. But on the other hand a single host could abuse the system taking a big percentage of that quota.

The `clientMaxRate` is a limit per client and it won't help you if you just want to control the total traffic, as
the total traffic supported by the backend or endpoint depends on the number of different requesting clients. A DDoS will then happily pass through, but on the other hand you can keep any particular abuser limited to its quota.

Depending on your use case you will need to decide if you use one, the other, the two, or none of them (fastest!)

