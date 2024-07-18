---
lastmod: 2023-02-16
old_version: true
date: 2016-07-01
linktitle: Endpoint Rate Limit
title: Rate Limiting API Gateway Endpoints
description: Implement rate limiting strategies in KrakenD API Gateway to control the number of requests and prevent API abuse or overloading
weight: 950
scope:
- endpoint
menu:
  community_v2.6:
    parent: "090 Traffic Management"
meta:
  since: 0.4
  source: https://github.com/krakend/krakend-ratelimit
  namespace:
  - qos/ratelimit/router
  scope:
  - endpoint
---

The router rate limit feature allows you to set the **maximum requests** a KrakenD endpoint will accept in a **given time window**. There are two different strategies to set limits that you can use separately or together:

- **Endpoint rate-limiting** (`max_rate`): applies simultaneously to all your customers using the endpoint, sharing the same counter.
- **User rate-limiting** (`client_max_rate`): applies to an individual user.

Both types keep **in-memory** an updated counter with the number of requests processed during the controlled time window in that endpoint.

For additional types of rate-limiting, see the [Traffic management overview](/docs/v2.6/throttling/).

## Configuration
The configuration allows you to use both types of rate limits (`max_rate` and `client_max_rate`) at the same time. For instance, let's set a limit of `50` requests `every` 10 minutes (`10m`), but every single user can do `5` every `10m`. A different IP equals a different user with this `strategy`:

```json
{
    "endpoint": "/limited-endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "max_rate": 50,
          "capacity": 50,
          "client_max_rate": 5,
          "client_capacity": 5,
          "every": "10m",
          "strategy": "ip"
        }
    }
}
```
{{< note title="Token Bucket" type="info" >}}
The rate limiting is based internally in the [Token Bucket algorithm](/docs/v2.6/throttling/token-bucket/). If you are unfamiliar, read the link to understand how it works.
{{< /note >}}



The following options are available to configure:

{{< schema version="v2.6" data="qos/ratelimit/router.json" >}}


## Endpoint rate-limiting (`max_rate`)
The endpoint rate limit acts on the number of simultaneous transactions an endpoint can process. This type of limit protects the service for all customers. In addition, these limits mitigate abusive actions such as rapidly writing content, aggressive polling, or excessive API calls.

It consumes a low amount of memory as it only needs one counter per endpoint.

When the users connected to an endpoint together exceed the `max_rate`, KrakenD starts to reject connections with a status code `503 Service Unavailable` and enables a [Spike Arrest](/docs/v2.6/throttling/spike-arrest/) policy.

Example:

```json
{
    "endpoint": "/endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "@comment":"A thousand requests every hour",
          "max_rate": 1000,
          "every": "1h"
        }
    }
}
```

## Client rate-limiting (`client_max_rate`)
The client or user rate limit applies to an individual user and endpoint. Each endpoint can have different limit rates, but all users are subject to the same rate.

{{< note title="A note on performance" >}}
Limiting endpoints per user makes KrakenD keep in-memory counters for the two dimensions: *endpoints x clients*.

The `client_max_rate` is more resource consuming than the `max_rate` as every incoming client needs individual tracking. Even though counters are space-efficient and very small in data, it's easy to end up with several millions of counters on big platforms.
{{< /note >}}

When a single user connected to an endpoint exceeds their `client_max_rate`, KrakenD starts to reject connections with a status code `429 Too Many Requests` and enables a [Spike Arrest](/docs/v2.6/throttling/spike-arrest/) policy.

Example:

```json
{
    "endpoint": "/endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "@comment":"20 requests every 5 minutes",
          "client_max_rate": 20,
          "every": "5m"
        }
    }
}
```

## Comparison of `max_rate` vs `client_max_rate`
The `max_rate` (also available as [proxy rate-limit](/docs/v2.6/backends/rate-limit/)) is an absolute number where you have exact control over how much traffic you are allowing to hit the backend or endpoint. In an eventual DDoS, the `max_rate` can help in a way since it won't accept more traffic than allowed. But on the other hand, a single host could abuse the system taking a significant percentage of that quota.

The `client_max_rate` is a limit per client, and it won't help you if you just want to control the total traffic, as the total traffic supported by the backend or endpoint depends on the number of different requesting clients. A DDoS will then happily pass through, but on the other hand, you can keep any particular abuser limited to its quota.

Depending on your use case, you must decide if you use one, the other, the two, or none of them.

## Playing together
You can set the two limiting strategies individually or together. Have in mind the following considerations:

- Setting the **client rate limit alone** can lead to a heavy load on your backends. For instance, if you have 200,000 active users in your platform at a given time and you allow each client ten requests per second (`client_max_rate : 10`), the permitted total traffic for the endpoint is: 200,000 users x 10 req/s = 2M req/s
- Setting the **endpoint rate limit alone** can lead to a single abuser limiting all other users in the platform.

So, in most cases, it is better to play them together.

### Rate-limiting by token claim
When you use rate-limiting with a `strategy` of `header`, you can set an arbitrary header name that will be used as the counter identifier. Then, when played in combination with JWT validation, you can extract values from the token and propagate them as new headers.

Propagated headers are available at the endpoint and backend levels, allowing you to set limits based on JWT criteria.

For instance, let's say you want to **rate-limit a specific department**, and your JWT token contains a claim `department`. You could have a configuration like this:

```json
{
    "endpoint": "/token-ratelimited",
    "input_headers": ["x-limit-department"],
    "extra_config": {
        "auth/validator": {
            "propagate_claims": [
                ["department","x-limit-department"]
            ]
        },
        "qos/ratelimit/router": {
            "client_max_rate": 100,
            "every": "1h",
            "strategy": "header",
            "key": "x-limit-department"
        }
    }
}
```

Notice that the `propagate_claims` in the validator adds the value of the claim `department` into a new header `x-limit-department`. The header is also added under `input_headers` because otherwise, the endpoint wouldn't see it ([zero-trust security](/docs/v2.6/design/zero-trust/)). Finally, the rate limit uses the new header as a strategy and specifies its name under `key`.

The department is now allowed to do `100` requests every `hour`.

### Examples of per-second rate limiting
The following examples demonstrate a configuration with several endpoints, each one setting different limits. As they don't set an `every` section, they will use the default of one second (`1s`):

- A `/happy-hour` endpoint with unlimited usage as it sets `max_rate = 0`
- A `/happy-hour-2` endpoint is equivalent to the previous one, as it has no rate limit configuration.
- A `/limited-endpoint` combines `client_max_rate` and `max_rate` together. It is capped at 50 reqs/s for all users, AND their users can make up to 5 reqs/s (where a user is a different IP)
- A `/user-limited-endpoint` is not limited globally, but every user (identified with `X-Auth-Token` can make up to 10 reqs/sec).

Configuration:

{{< highlight json "hl_lines=7-9 31-35 41-45">}}
{
  "version": 3,
  "endpoints": [
    {
        "endpoint": "/happy-hour",
        "extra_config": {
            "qos/ratelimit/router": {
                "max_rate": 0,
                "client_max_rate": 0
            }
        },
        "backend": [
          {
            "url_pattern": "/__health",
            "host": ["http://localhost:8080"]
          }
        ]
    },
    {
        "endpoint": "/happy-hour-2",
        "backend": [
          {
            "url_pattern": "/__health",
            "host": ["http://localhost:8080"]
          }
        ]
    },
    {
        "endpoint": "/limited-endpoint",
        "extra_config": {
          "qos/ratelimit/router": {
              "max_rate": 50,
              "client_max_rate": 5,
              "strategy": "ip"
            }
        }
    },
    {
        "endpoint": "/user-limited-endpoint",
        "extra_config": {
          "qos/ratelimit/router": {
              "client_max_rate": 10,
              "strategy": "header",
              "key": "X-Auth-Token"
            }
        },
        "backend": [
          {
            "url_pattern": "/__health",
            "host": ["http://localhost:8080"]
          }
        ]
    }
{{< /highlight >}}

### Examples of per-minute or per-hour rate limiting
The rate limit component measures the router activity using the time window selected under `every`. You can use hours or minutes instead of seconds or you could even set daily or monthly rate-limiting, but taking into account that the counters reset every time you deploy the configuration.

To use units larger than an hour, just express the days by hours. Using large units is not convenient if you often deploy (unless you use the persisted [Redis rate limit {{< badge color="denim" >}}Enterprise{{< /badge >}}
](/docs/enterprise/throttling/global-rate-limit/))

For example, let's say you want the endpoint to cut the access at `30 reqs/day`. It means that within a day, whether the users exhaust the 30 requests in one second or gradually across the day, you won't let them do more than `30` every day. So how do we apply this to the configuration?

The configuration would be:

```json
{
  "qos/ratelimit/router": {
    "@comment": "Client rate limit of 30 reqs/day",
    "client_max_rate": 30,
    "client_capacity": 30,
    "every": "24h"
  }
}
```

Similarly, 30 requests every 5 minutes, could be set like this.

```json
{
  "qos/ratelimit/router": {
    "@comment": "Endpoint rate limit of 30 reqs/hour",
    "max_rate": 30,
    "every": "5m",
    "capacity": 30
  }
}
```

In summary, the `client_max_rate` and the `max_rate` set the speed at which you refill new usage tokens to the user. On the other hand, the `capacity` and `client_capacity` let you play with the buffer you give to the users and let them spend 30 requests in a single second (within the 5 minutes) or not.

For more information, see the [Token Bucket algorithm](/docs/enterprise/throttling/token-bucket/).