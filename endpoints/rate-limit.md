---
lastmod: 2024-06-14
date: 2016-07-01
linktitle: Endpoint Rate Limit
title: Rate Limiting API Gateway Endpoints
description: Implement rate-limiting strategies in KrakenD API Gateway to control the number of requests and prevent API abuse or overloading
weight: 950
scope:
- endpoint
menu:
  community_current:
    parent: "090 Traffic Management"
meta:
  since: 0.4
  source: https://github.com/krakend/krakend-ratelimit
  namespace:
  - qos/ratelimit/router
  scope:
  - endpoint
---

The router rate limit feature allows you to set the **maximum requests** a KrakenD endpoint (a route) will accept in a **given time window**. There are two different strategies to set limits that you can use separately or together:

- **Endpoint rate-limiting** (`max_rate`): applies simultaneously to all clients using the endpoint, sharing a unique counter.
- **User rate-limiting** (`client_max_rate`): sets a counter to each individual user.

Both types can coexist and they complement each other, and store the counters **in-memory**. On a cluster, each machine sees and counts only its passing traffic.

There are [additional types of rate-limiting](/docs/throttling/).

{{< note title="Token Bucket" type="info" >}}
The rate limiting is based internally on the [Token Bucket algorithm](/docs/throttling/token-bucket/). If you are unfamiliar with it, read the link to understand how it works.
{{< /note >}}

## Comparing `max_rate` and `client_max_rate`
Imagine you have Mary and Fred using your API, and they connect to an endpoint `/v1/checkout/payment` that you want to rate-limit. If you add a `max_rate`, you limit the activity they generate together. It does not matter who is making more or fewer requests; the endpoint will be inaccessible for everyone once the throughput surpasses the limit you have set.

On the other hand, adding the `client_max_rate` monitors Fred and Mary's activity separately. If one of them is an abuser, the access is cut, while the other can continue to use the endpoint.

The `max_rate` (also available as [proxy rate-limit](/docs/backends/rate-limit/)) is an absolute number that gives you exact control over how much traffic you allow to hit the backend or endpoint. In an eventual DDoS, the `max_rate` can help in a way since it won't accept more traffic than allowed. On the other hand, a single host could abuse the system by taking a significant percentage of that quota.

The `client_max_rate` is a limit per client, and it won't help you if you just want to control the total traffic, as the total traffic supported by the backend or endpoint depends on the number of different requesting clients. A DDoS will then happily pass through, but you can keep any particular abuser limited to its quota.

As we said, you can set the two limiting strategies individually or together. Have in mind the following considerations:

- Setting the **client rate limit alone** on a platform with many users can lead to a heavy load on your backends. For instance, if you have 200,000 active users in your platform at a given time and you allow each client ten requests per second (`client_max_rate: 10`, `every: 1s`), the permitted total traffic for the endpoint is 200,000 users x 10 req/s = 2M req/s
- Setting the **endpoint rate limit alone** can lead to a single abuser limiting all other users in the platform.

So, in most cases, it is better to play them together. Adding also a [Circuit Breaker](/docs/backends/circuit-breaker/) is even better.


## Configuration
The `max_rate` and `client_max_rate` configurations are under a common namespace, `qos/ratelimit/router` (QoS stands for Quality of Service).

For instance, let's start with a simple and mixed example that sets two limits:
- `50` requests every `10m` (10 minutes) among all clients
- `5` requests per client every `10m`.

```json
{
    "endpoint": "/limited-endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "max_rate": 50,
          "every": "10m",
          "client_max_rate": 5
      }
    }
}
```

An expanded and **more explicit configuration** that represents the same idea would be:

```json
{
    "endpoint": "/limited-endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "max_rate": 50,
          "every": "10m",
          "client_max_rate": 5,

          "strategy": "ip",
          "capacity": 50,
          "client_capacity": 5
      }
    }
}
```
In this configuration, we have set the IP `strategy`, which considers that every IP accessing the gateway is a different client. However, a client could be a JWT token, a header, or even a parameter.

We have also set the `capacity` for the `max_rate` and the `client_capacity` for the `client_max_rate`, which sets the maximum buffer for any given instant. See below.

## Endpoint rate-limiting (`max_rate`)
The endpoint rate limit acts on the number of simultaneous transactions an endpoint can process. This type of limit protects the service for all customers. In addition, these limits mitigate abusive actions such as rapidly writing content, aggressive polling, or excessive API calls.

It consumes low memory as it only needs one counter per endpoint.

When the users connected to an endpoint together exceed the `max_rate`, KrakenD starts to reject connections with a status code `503 Service Unavailable` and enables a [Spike Arrest](/docs/throttling/spike-arrest/) policy.

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

{{< schema data="qos/ratelimit/router.json" filter="max_rate,capacity,every" title="Endpoint rate limit options" >}}

## Client rate-limiting (`client_max_rate`)
The client or user rate limit applies one counter to each individual user and endpoint. Each endpoint can have different limit rates, but all users are subject to the same rate.

{{< note title="A note on performance" >}}
Limiting endpoints per user makes KrakenD keep in-memory counters for the two dimensions: *endpoints x clients*.

The `client_max_rate` is more resource-consuming than the `max_rate` as every incoming client needs individual tracking. Even though counters are space-efficient and very small in data, many endpoints with many concurrent users will lead to higher memory consumption.
{{< /note >}}

When a single user connected to an endpoint exceeds their `client_max_rate,` KrakenD starts rejecting connections with a status code `429 Too Many Requests` and enables a [Spike Arrest](/docs/throttling/spike-arrest/) policy.

Each client's counter is stored in memory only for the time needed to deliver the traffic restriction properly. The needed time is calculated automatically based on your configuration, and we call this time the **TTL**. A specific routine (or more) deletes outdated counters during runtime. See micro-optimizations below for more details.

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
The following configuration options are specific to the client rate limiting:

{{< schema data="qos/ratelimit/router.json" filter="client_max_rate,client_capacity,every,strategy,key" title="Client rate limit options" >}}

Below, you'll see different interpretations of what a client is.

### Client rate-limiting by token claim
Setting a rate limit for every issued token could be as easy as:

```json
{
  "endpoint": "/foo",
  "extra_config": {
    "auth/validator": {
      "@comment": "Omitted for simplicity"
    },
    "qos/ratelimit/router": {
      "client_max_rate": 100,
      "every": "1h",
      "strategy": "header",
      "key": "Authorization"
    }
  }
}
```
The endpoint now limits to 100 requests per hour to every different valid token (valid because the [JWT validator](/docs/authorization/jwt-validation/) takes care of it).

But instead of rate limiting based on the whole token, you can also rate limit based on claims of the token by propagating claims as headers. For instance, let's say you want to **rate-limit a specific department**, and your JWT token contains a claim `department`.

If you have token validation and use the client rate-limiting with a `strategy` of `header`, you can set an arbitrary header name for the counter identifier. Propagated headers are available at the endpoint and backend levels, allowing you to set limits based on JWT criteria.

You could have a configuration like this:

```json
{
  "endpoint": "/token-ratelimited",
  "input_headers": [
    "x-limit-department"
  ],
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

Notice that the `propagate_claims` in the validator adds the claim `department` value into a new header, `x-limit-department`. The header is also added under `input_headers` because otherwise, the endpoint wouldn't see it ([zero-trust security](/docs/design/zero-trust/)). Finally, the rate limit uses the new header as a strategy and specifies its name under `key`.

The department can now do `100` requests every `hour`. You can extrapolate this to any other claim, like the subject or anything else you need.

### Rate-limiting by URL parameter
A different case of the `client_max_rate` is when used with the `strategy` equal to `param`. Instead of limiting a specific user (through a token, a claim, or a header), you consider that the client comes in the URL as a parameter. For instance, you provide an API containing endpoints like `/api/{customer_id}/invoices`, and you want to consider that every different `customer_id` is a different client.

In that case, you can rate limit the parameter of the endpoint as follows:

```json
{
  "endpoint": "/api/{customer_id}/invoices",
  "extra_config": {
    "qos/ratelimit/router": {
      "client_max_rate": 5,
      "every": "1m",
      "strategy": "param",
      "key": "customer_id"
    }
  }
}
```

The configuration above would allow 5 requests to `/api/1234/invoices` every minute and another 5 to `/api/5678/invoices`. In a scenario like this, it would be advisable that you add a [security policy](/docs/enterprise/security-policies/) {{< badge >}}Enterprise{{< /badge >}}
 that makes sure clients cannot abuse the rate limits of others.

### Micro-optimizations of the client_rate_limit
There are a few advanced values that you can add to the rate limit if you want to fine-tune CPU and Memory consumption. These values are not needed in most of the cases, but the door is open to tune how the rate limit works internally.

{{< schema data="qos/ratelimit/router.json" filter="num_shards,cleanup_period,cleanup_threads" title="Micro-optimization of rate limiting" >}}

Example:

```json
{
  "endpoint": "/api/invoices",
  "extra_config": {
    "qos/ratelimit/router": {
      "client_max_rate": 5,
      "every": "1m",
      "num_shards": 2048,
      "cleanup_period": "60s",
      "cleanup_threads": 1
    }
  }
}
```

### Examples of per-second rate limiting
The following examples demonstrate a configuration with several endpoints, each one setting different limits. As they don't set an `every` section, they will use the default of one second (`1s`):

- A `/happy-hour` endpoint with unlimited usage as it sets `max_rate = 0`
- A `/happy-hour-2` endpoint is equivalent to the previous one, as it has no rate limit configuration.
- A `/limited-endpoint` combines `client_max_rate` and `max_rate` together. It is capped at 50 reqs/s for all users, AND their users can make up to 5 reqs/s (where a user is a different IP)
- A `/user-limited-endpoint` is not limited globally, but every user (identified with `X-Auth-Token` can make up to 10 reqs/sec).

Configuration:

```json
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
          "host": [
            "http://localhost:8080"
          ]
        }
      ]
    },
    {
      "endpoint": "/happy-hour-2",
      "backend": [
        {
          "url_pattern": "/__health",
          "host": [
            "http://localhost:8080"
          ]
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
          "host": [
            "http://localhost:8080"
          ]
        }
      ]
    }
  ]
}
```

### Examples of per-minute or per-hour rate limiting
The rate limit component measures the router activity using the time window selected under `every`. You can use hours or minutes instead of seconds or you could even set daily or monthly rate-limiting, but taking into account that the counters reset every time you deploy the configuration.

To use units larger than an hour, just express the days by hours. Using large units is not convenient if you often deploy (unless you use the persisted [Redis rate limit {{< badge >}}Enterprise{{< /badge >}}
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
