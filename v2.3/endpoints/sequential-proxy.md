---
lastmod: 2022-11-24
old_version: true
date: 2018-11-11
toc: true
linktitle: Sequential Proxy (chain reqs.)
title: Sequential Proxy
since: 0.7
notoc: true
weight: 60
images:
- /images/documentation/krakend-sequential-call.png
menu:
  community_v2.3:
    parent: "040 Endpoint Configuration"
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
The best experience consumers can have with KrakenD API is by letting the system fetch all the data from the different backends concurrently at the same time. However, there are times when you need to **delay a backend call** until you can inject as input the result of a previous call.

The sequential proxy allows you to **chain backend requests**.

{{< note title="Chained calls are considered an anti-pattern" type="info" >}}
Making use of sequential calls is considered an anti-pattern. This is because when you make a service dependent on the other, you are increasing the latency of the service, decreasing its performance, and augmenting its error rate.

For instance, if you have three backends with an error rate of 10% each when calling them in series, it produces an error rate of 27%.
{{< /note >}}

## Chaining the requests
All you need to enable the sequential proxy is add in the endpoint definition the following configuration:

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


When the sequential proxy is enabled, the `url_pattern` of every backend can use a new variable that references the **resp**onse of a previous API call. The variable has the following construction:

```js
{resp0_XXXX}
```

Where `0` is the index of the specific `backend` you want to access (`0` is the first backend), and where `XXXX` is the attribute name you want to inject from the response of the previous call. You can access **nested objects** of the response using the dot notation, for example `{resp0_user.hash}` will retrieve the value `abcdef` from the response `{"user": { "hash": "abcdef }}`. **You cannot access nested objects inside arrays or collections, they must be objects**.

{{< note title="Unsafe operations" >}}
If you use unsafe methods (not a `GET`), they can only be placed in the last position of the sequence. Sequences are meant to be used in read-only operations except for the last call. A sequence is not meant to be used in distributed transactions.
{{< /note >}}

If the encoding of your backend is `string`, then you can access its contents using `resp0_content`.

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
