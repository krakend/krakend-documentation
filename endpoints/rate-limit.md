---
lastmod: 2022-01-28
date: 2016-07-01
linktitle: Rate Limits
title: Router Rate-limiting
weight: 10
scope:
- endpoint
menu:
  community_current:
    parent: "040 Endpoint Configuration"
meta:
  since: 0.4
  source: https://github.com/krakendio/krakend-ratelimit
  namespace:
  - qos/ratelimit/router
  scope:
  - endpoint
---

The router rate limit feature allows you to set a number of **maximum requests per second** a KrakenD endpoint will accept. There are two different strategies to set limits that you can use separately or together:

- **Endpoint rate-limiting**: applies simultaneously to all your customers using the endpoint, sharing the same counter.
- **User rate-limiting**: applies to an individual user.

Both types keep **in-memory** an updated counter with the number of requests processed per second in that endpoint.

For additional types of rate-limiting see the [Traffic management overview](/docs/throttling/overview/).

## Endpoint rate-limiting (`max_rate`)
The endpoint rate limit acts on the number of simultaneous transactions an endpoint can process. This type of limit protects the service for all customers. In addition, these limits mitigate abusive actions such as rapidly writing content, aggressive polling, or excessive API calls.

It consumes a low amount of memory as it only needs one counter per endpoint.

When the users connected to an endpoint together exceed the `max_rate`, KrakenD starts to reject connections with a status code `503 Service Unavailable` and enables a [Spike Arrest](/docs/throttling/spike-arrest/) policy

## Client rate-limiting (`client_max_rate`)
The client or user rate limit applies to an individual user and endpoint. Each endpoint can have different limit rates, but all users are subject to the same rate.

{{< note title="A note on performance" >}}
Limiting endpoints per user makes KrakenD keep in-memory counters for the two dimensions: *endpoints x clients*.

The `client_max_rate` is less performant than the `max_rate` as every incoming client needs individual tracking. Even that counters are efficient and very small in data, it's easy to end up with several millions of counters on big platforms. Make sure to do your math.
{{< /note >}}
When a single user connected to an endpoint exceeds their `client_max_rate`, KrakenD starts to reject connections with a status code `429 Too Many Requests` and enables a [Spike Arrest](/docs/throttling/spike-arrest/) policy

## Playing together
You can set the two limiting strategies individually or together. Have in mind the following considerations:

- Setting the **client rate limit alone** can lead to a heavy load of your backends. For instance, if you have 200,000 active users in your platform at a given time and you allow each client 10 requests per second (`client_max_rate : 10`) the permitted total traffic for the endpoint is: 200,000 users x 10 req/s = 2M req/s
- Setting the **endpoint rate limit alone** can lead to a single abuser limiting all other users in the platform.

So, in most of the cases, is better to play together.

## Configuration
The configuration allows you to use both types of rate limits at the same time:

{{< highlight json>}}
{
    "endpoint": "/limited-endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "max_rate": 50,
          "client_max_rate": 5,
          "strategy": "ip"
        }
    }
}
{{< /highlight >}}

The following options are available to configure. You can use `max_rate` and `client_max_rate` together or separated.

- `max_rate` (*integer*): Sets the number of **maximum requests the endpoint can handle per second**. The absence of `max_rate` in the configuration or `0` is the equivalent to no limitation.
- `client_max_rate` (*integer*): Number of requests per second this endpoint will accept for each user (*user quota*). The client is defined by `strategy`. Instead of counting all the connections to the endpoint as the option above, the `client_max_rate` keeps a counter for every client and endpoint. Keep in mind that every KrakenD instance keeps its counters in memory for **every single client**.
- `strategy` (*string*): The strategy you will use to set client counters. One of `ip` or `header`. Only to be used in combination with `client_max_rate`.
  - `"strategy": "ip"` When the restrictions apply to the client's IP, and every IP is considered to be a different user. Optionally a `key` can be used to extract the IP from a custom header:
    - E.g, set `"key": "X-Original-Forwarded-For"` to extract the IP from a header containing a list of space-separated IPs (will take the first one).
  - `"strategy": "header"` When the criteria for identifying a user comes from the value of a `key` inside the header. With this strategy, the `key` must also be present.
    - E.g., set `"key": "X-TOKEN"` to use the `X-TOKEN` header as the unique user identifier.



### Example
The following example demonstrates a configuration with several endpoints, each one setting different limits:

- A `/happy-hour` endpoint with unlimited usage as it sets `max_rate = 0`
- A `/happy-hour-2` endpoint is equivalent to the previous, as it has no rate limit configuration.
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
