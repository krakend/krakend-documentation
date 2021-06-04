---
lastmod: 2021-05-02
date: 2016-07-01
linktitle: Rate Limits
title: Endpoint rate limiting
weight: 10
scope:
- endpoint
menu:
  community_current:
    parent: "040 Endpoint Configuration "
meta:
  since: 0.4
  source: https://github.com/devopsfaith/krakend-ratelimit
  namespace:
  - github.com/devopsfaith/krakend-ratelimit/juju/router
  scope:
  - endpoint
---

Limiting endpoints is the responsibility of the **router rate** and allows you to set the number of **maximum requests per second** a KrakenD endpoint will accept. By default, there is no limitation on the number of requests an endpoint can handle.

To specify a rate limit, you need to add the configuration in the desired endpoint.

At the router level, you can set the rate limit for endpoints based on:

1. **Endpoint rate limit** (`maxRate`): Maximum number of requests an endpoint accepts in a second, no matter where the traffic comes from.
2. **Client/User rate limit** (`clientMaxRate`): Maximum number of requests an endpoint accepts **per client**

When any of these strategies are set, every KrakenD instance keeps **in-memory** an updated counter with the number of requests processed per second in that endpoint. 
## Configuration

{{< highlight json>}}
{
    "endpoint": "/limited-endpoint",
    "extra_config": {
      "github.com/devopsfaith/krakend-ratelimit/juju/router": {
          "maxRate": 50,
          "clientMaxRate": 5,
          "strategy": "ip"
        }
    }
}
{{< /highlight >}}

The following options are available to configure. You can use `maxRate` and `clientMaxRate` together or sepparated. 

- `maxRate` (*integer*): Sets the number of **maximum requests the endpoint can handle per second**. The absence of `maxRate` in the configuration or `0` is the equivalent to no limitation.
- `clientMaxRate` (*integer*): Number of requests per second this endpoint will accept for each user (*user quota*). The client is defined by `strategy`. Instead of counting all the connections to the endpoint as the option above, the `clientMaxRate` keeps a counter for every client and endpoint. Keep in mind that every KrakenD instance keeps its counters in memory for **every single client**. 
- `strategy` (*string*): The strategy you will use to set client counters. One of `ip` or `header`. Only to be used in combination with `clientMaxRate`.


### Client identification strategies
Two ways of identifiying a client are available:

  - `"strategy": "ip"` When the restrictions apply to the client's IP, and every IP is considered to be a different user. Optionally a `key` can be used to extract the IP from a custom header:
    - E.g, set `"key": "X-Original-Forwarded-For"` to extract the IP from a header containing a list of space-separated IPs (will take the first one). 
  - `"strategy": "header"` When the criteria for identifying a user comes from the value of a `key` inside the header. With this strategy, the `key` must also be present.
    - E.g., set `"key": "X-TOKEN"` to use the `X-TOKEN` header as the unique user identifier.

## Rate limit status codes
KrakenD rejects with a specific HTTP status code all requests above the limit set:

- `503 Service Unavailable` if the `maxRate` limit is reached to whoever triggered the limit. 
- `429 Too Many Requests` if the `clientMaxRate` limit is reached by a specific user (others who didn't will continue using the endpoint normally).

## Considerations
The two limiting strategies can be set individually or together. Have in mind the following considerations:

- Setting the **client rate limit alone** can lead to a heavy load of your backends. 
- Setting the **endpoint rate limit alone** can lead to a single abuser limiting all other users in the platform.

For instance, if you have 200,000 active users in your platform at a given time and you allow each client 10 requests per second (`clientMaxRate : 10`) the total allowed traffic for the endpoint is:

    200,000 users x 10 req/s = 2M req/s

{{< note title="A note on performance" >}}
Limiting endpoints per user makes KrakenD keep in memory counters for the two dimensions: *endpoints x clients*.

The `clientMaxRate` is less performant than the `maxRate` as every incoming client needs individual tracking. Even that counters are efficient and very small in data, it's easy to end up with several millions of counters on big platforms. Make sure to do your math.
{{< /note >}}

### Example
The following example demonstrates a configuration with several endpoints, each one setting different limits:

- A `/happy-hour` endpoint with unlimited usage as it sets `maxRate = 0`
- A `/happy-hour-2` endpoint is equivalent to the previous, as it has no rate limit configuration.
- A `/limited-endpoint` combines `clientMaxRate` and `maxRate` together. It is capped at 50 reqs/s for all users, AND their users can make up to 5 reqs/s (where a user is a different IP)
- A `/user-limited-endpoint` is not limited globally, but every user (identified with `X-Auth-Token` can make up to 10 reqs/sec). 

Configuration:

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
              "endpoint": "/happy-hour-2"
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
              }
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
