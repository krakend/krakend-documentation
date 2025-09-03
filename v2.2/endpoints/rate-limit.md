---
lastmod: 2023-02-16
old_version: true
date: 2016-07-01
linktitle: Rate Limits
title: Router Rate-limiting
weight: 10
scope:
- endpoint
menu:
  community_v2.2:
    parent: "040 Endpoint Configuration"
meta:
  since: v0.4
  source: https://github.com/krakend/krakend-ratelimit
  namespace:
  - qos/ratelimit/router
  scope:
  - endpoint
---

The router rate limit feature allows you to set the **maximum requests per second** (convertible from minutes or hours, too) a KrakenD endpoint will accept. There are two different strategies to set limits that you can use separately or together:

- **Endpoint rate-limiting**: applies simultaneously to all your customers using the endpoint, sharing the same counter.
- **User rate-limiting**: applies to an individual user.

Both types keep **in-memory** an updated counter with the number of requests processed per second in that endpoint.

For additional types of rate-limiting, see the [Traffic management overview](/docs/v2.2/throttling/).

## Endpoint rate-limiting (`max_rate`)
The endpoint rate limit acts on the number of simultaneous transactions an endpoint can process. This type of limit protects the service for all customers. In addition, these limits mitigate abusive actions such as rapidly writing content, aggressive polling, or excessive API calls.

It consumes a low amount of memory as it only needs one counter per endpoint.

When the users connected to an endpoint together exceed the `max_rate`, KrakenD starts to reject connections with a status code `503 Service Unavailable` and enables a [Spike Arrest](/docs/v2.2/throttling/spike-arrest/) policy

## Client rate-limiting (`client_max_rate`)
The client or user rate limit applies to an individual user and endpoint. Each endpoint can have different limit rates, but all users are subject to the same rate.

{{< note title="A note on performance" >}}
Limiting endpoints per user makes KrakenD keep in-memory counters for the two dimensions: *endpoints x clients*.

The `client_max_rate` is heavier than the `max_rate` as every incoming client needs individual tracking. Even though counters are efficient and very small in data, it's easy to end up with several millions of counters on big platforms. So make sure to do your math.
{{< /note >}}

When a single user connected to an endpoint exceeds their `client_max_rate`, KrakenD starts to reject connections with a status code `429 Too Many Requests` and enables a [Spike Arrest](/docs/v2.2/throttling/spike-arrest/) policy

## Playing together
You can set the two limiting strategies individually or together. Have in mind the following considerations:

- Setting the **client rate limit alone** can lead to a heavy load on your backends. For instance, if you have 200,000 active users in your platform at a given time and you allow each client ten requests per second (`client_max_rate : 10`), the permitted total traffic for the endpoint is: 200,000 users x 10 req/s = 2M req/s
- Setting the **endpoint rate limit alone** can lead to a single abuser limiting all other users in the platform.

So, in most cases, it is better to play them together.

## Configuration
The configuration allows you to use both types of rate limits at the same time:

```json
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
```

The following options are available to configure. You can use `max_rate` and `client_max_rate` together or separately. The rate limiting uses the [Token Bucket algorithm](/docs/v2.2/throttling/token-bucket/). The `capacity` (or `client_capacity`) of the bucket is, by default, equal to the maximum rate, but you might want to set a different value when not using seconds as a measurement unit or when you want a separate buffer.

{{< schema version="v2.2" data="qos/ratelimit/router.json" >}}

### Rate-limiting by token claim
When you use rate-limiting with a `strategy` of `header`, you can set an arbitrary header name that will be used as the counter identifier. Then, when played in combination with JWT validation, you can extract values from the token and propagate them as new headers.

Propagated headers are available at the endpoint and backend levels, allowing you to set limits based on JWT criteria. For instance, let's say you want to rate-limit a specific department, and your JWT token contains a claim `department`. You could have a configuration like this:
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
            "max_rate": 50,
            "client_max_rate": 5,
            "strategy": "header",
            "key": "x-limit-department"
        }
    }
}
```

Notice that the `propagate_claims` in the validator adds the department value into a new header, `x-limit-department`. The header is also added under `input_headers` because otherwise, the endpoint wouldn't see it (zero-trust policy). Finally, the rate limit uses the new header as a strategy and specifies its name under `key`.

### Examples of per-second rate limiting
The following examples demonstrate a configuration with several endpoints, each one setting different limits:

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
The rate limit component measures the router activity using the seconds unit. Nevertheless, you can set rate limits on larger time units, like minutes or hours, and you only need to divide the desired unit to express into seconds.

You could go even to daily or monthly rate-limiting, but taking into account that the counters reset every time you deploy the configuration, using large units is not convenient if you often deploy (unless you use the persisted [Redis rate limit {{< badge >}}Enterprise{{< /badge >}}
](/docs/enterprise/throttling/global-rate-limit/))

For example, let's say you want the endpoint to cut the access at `30 reqs/minute`. It means that within a minute, whether the users exhaust the 30 requests in one second or gradually across the minute, you won't let them do more than `30` every minute on average. So how do we apply this to the configuration?

Depending on the type of rate limit you want to apply (endpoint rate limit or client rate limit), you will need to set a `capacity` (or a `client_capacity`) first.

Whether the end-user has a sustained activity across the minute or it quickly uses all the credit in the early seconds of the minute, you don't want to go over an average of `30 reqs` every minute. The `capacity` settings let you specify the maximum number of requests you can instantly consume. In this case, the capacity could be `30`, although you can set it differently.

Then as the user drains the credits, the rate limit will add more credits into the system at a rate of `30req / 60 seconds = 0.5 req/sec`.

The configuration would be:

```json
{
  "qos/ratelimit/router": {
    "@comment": "Client rate limit of 30 reqs/minute",
    "client_max_rate": 0.05,
    "client_capacity": 30
  }
}
```

Similarly, `30 reqs/hour` for an endpoint rate limit, instead of a client rate limit, would be the operation `30reqs / 60minutes / 60secs = 0.008333333`

```json
{
  "qos/ratelimit/router": {
    "@comment": "Endpoint rate limit of 30 reqs/hour",
    "max_rate": 0.008333333,
    "capacity": 30
  }
}
```

In summary, the `client_max_rate` and the `max_rate` set the speed at which you refill new usage tokens to the user. On the other hand, the `capacity` and `client_capacity` let you play with the buffer you give to the users and let them spend 30 requests in a single second (within the minute) or not.

For more information, see the [Token Bucket algorithm](/docs/enterprise/throttling/token-bucket/).