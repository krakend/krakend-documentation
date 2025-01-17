---
lastmod: 2023-10-15
old_version: true
date: 2018-11-11
linktitle: Sequential Proxy (chain reqs.)
title: Sequential Proxying
description: Explore the sequential proxying capability in KrakenD API Gateway, allowing you to chain multiple requests and orchestrate complex API workflows
since: 0.7
notoc: false
weight: 80
images:
- /images/documentation/krakend-sequential-call.png
menu:
  community_v2.8:
    parent: "040 Routing and Forwarding"
meta:
  noop_incompatible: true
  since: 0.7
  source: https://github.com/luraproject/lura
  namespace:
  - proxy
  scope:
  - endpoint
  - async_agent
  log_prefix:
  - "[SERVICE: Gin]"
---
The best experience consumers can have with KrakenD API is by letting the system fetch all the data from the different backends simultaneously. However, sometimes you need to **delay a backend call** until you have called a previous service. Although this is not ideal, the sequential proxy allows you to **chain backend requests**.

## Do you really need a sequential proxy?
{{< note title="Chained calls are considered an anti-pattern" type="warning" >}}{{< /note >}}

Using sequential calls is considered an anti-pattern because when you make a network service dependent on the other, you are **increasing the latency, decreasing the performance, and augmenting the error rate**.

It is much worse than using [aggregation](/docs/v2.8/endpoints/response-manipulation/#aggregation-and-merging). In aggregation, parallel requests execute simultaneously, whereas, in sequential aggregation, requests are executed one at a time, with each one waiting for the previous request to finish before moving on to the next. If a backend in a sequence fails, the process aborts, and the next backend is never reached, so there are many more chances that your users are left without data.

From an error rate perspective, the nature of sequential proxy performs more deficient: Suppose you have three backends with an error rate of 10% each, then the probability of success separately in each is 90%. But when executing the three of them sequentially, the success rate drops to 73% (because `0.9 * 0.9 * 0.9 = 0.729`).

In an aggregation scenario, the probability of having at least one working call is the opposite of the likelihood of having all calls result in errors. So, the chance of all three calls resulting in errors is 0.1% (because `0.1 * 0.1 * 0.1 = 0.001`).

The contrast between a 99.9% chance of some data availability and a 73% probability is quite substantial. Isn't it? That being said, from an architectural point of view, the sequential proxy should be your last resort.

In addition, since KrakenD 2.5, you can add to a sequence **multiple unsafe methods** (methods different than `GET`). When you chain several write requests in multiple nodes, you execute a **distributed transaction** in a flowery disguise, as in a database. But a gateway is not a database, and you don't have any rollback mechanism if one of your write methods fails, so you can only *hope for the best*.

## Sequential proxy configuration
To enable the sequential proxy, you need to add in the endpoint definition the following configuration:

```json
{
    "endpoint": "/hotels/{id}",
    "extra_config": {
          "proxy": {
              "sequential": true
          }
      }
}
```

After doing this, the list of `backend` is executed one by one, and the next call has access to the data returned by the previous in the `url_pattern` and not anywhere else. **The body of the last request is not sent to the next** (but you can still access the original body).

When the sequential proxy is enabled, the `url_pattern` of every backend can use a new variable that references the **resp**onse of a previous API call. The variable has the following construction:

```js
{resp0_XXXX}
```

Where `0` is the index of the specific `backend` you want to access (`0` is the first backend), and where `XXXX` is the attribute name you want to inject from the response of the previous call. You can access **nested objects** of the response using the dot notation. For example, given a response `{"user": { "hash": "abcdef }}`, the variable`{resp0_user.hash}` will contain the value `abcdef`. **You cannot access nested objects inside arrays or collections**: responses must be objects.

If the encoding of your backend is `string`, then you can access its contents using `resp0_content`.

It does not matter if the `{resp0_XXXX}` variable is part of the URL or if it is passed as a query string. For instance, the following examples would work:

```json
{
    "url_pattern": "/user/{resp0_user.hash}"
}
```

And also:

```json
{
    "url_pattern": "/user?hash={resp0_user.hash}"
}
```

## Example
It's easier to understand with the example of the graph:

![Chained call](/images/documentation/krakend-sequential-call.png)

The user calls the gateway with an URL like `/hotel-destinations/{id}`, which needs to fetch the hotel information and all its associated destinations. Let's say the ID they request is `25`. The gateway calls a backend `/hotels/25` that returns data for the requested hotel, including a `destination_id` field that is a relationship identifier. The output for `GET /hotels/25` is like the following:

```json
{
    "hotel_id": 25,
    "name": "Hotel California",
    "destination_id": 1034
}
```

KrakenD waits for the backend response and injects the value of `destination_id` in the URL of the next backend call. In this case, the next call is `GET /destinations/1034`, and the response is:

```json
{
    "destination_id": 1034,
    "destinations": [
        "LAX",
        "SFO",
        "OAK"
    ]
}
```

Now KrakenD has both responses from the backends and can merge the data, returning the following aggregated object to the user:

```json
{
    "hotel_id": 25,
    "name": "Hotel California",
    "destination_id": 1034,
    "destinations": [
        "LAX",
        "SFO",
        "OAK"
    ]
}
```

The configuration needed for this example is:

```json
{
    "endpoint": "/hotel-destinations/{id}",
    "backend": [
        {
            "@comment": "This is the index position 0",
            "host": [
                "https://hotels.api"
            ],
            "url_pattern": "/hotels/{id}"
        },
        {
            "@comment": "This is the index position 1",
            "host": [
                "https://destinations.api"
            ],
            "@comment2": "resp0_ is the response of index position 0",
            "url_pattern": "/destinations/{resp0_destination_id}"
        }
    ],
    "extra_config": {
        "proxy": {
            "sequential": true
        }
    }
}
```

The key here is the variable `{resp0_destination_id}` that refers to `destination_id` for the backend with index `0` (first in the list).
