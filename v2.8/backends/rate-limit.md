---
lastmod: 2023-06-23
old_version: true
date: 2018-10-29
linktitle: Proxy rate limit
title: Rate Limiting  Backends
description: Implement rate limiting in KrakenD API Gateway backends to control and manage API usage, preventing abuse and ensuring fair resource allocation
weight: 960
menu:
  community_v2.8:
    parent: "090 Traffic Management"
notoc: true
meta:
  since: false
  source: https://github.com/krakend/krakend-ratelimit
  namespace:
  - qos/ratelimit/proxy
  scope:
  - backend
---

No matter what amount of activity the users generate at the router level, you can limit KrakenD's connections to your backends. The configuration is similar to the [router's rate limit](/docs/v2.8/endpoints/rate-limit/), but it's declared directly in the `backend` section instead of the `endpoint`.

The limit applies **per defined backend entry** and does not consider the activity other backends generate. Each `backend` entry handles its counters and does not share them with different backends or endpoints.

The proxy rate limit is defined in the `krakend.json` configuration file as follows:

{{< highlight json "hl_lines=7-12">}}
{
    "endpoint": "/products/{cat_id}",
    "backend": [{
        "host": ["http://some.api.com/"],
        "url_pattern": "/catalog/category/{cat_id}.rss",
        "encoding": "rss",
        "extra_config": {
            "qos/ratelimit/proxy": {
                "max_rate": 0.5,
                "every": "1s",
                "capacity": 1
            }
        }
    }]
}
{{< /highlight >}}

These are the parameters you can set:

{{< schema version="v2.8" data="qos/ratelimit/proxy.json" >}}

## Comparison with router rate limit
In a nutshell:

- The [router rate limit](/docs/v2.8/endpoints/rate-limit/) controls the requests users do to KrakenD.
- The proxy rate limit controls KrakenD's requests to your services.

You don't have to choose one or the other; you can mix the different types as they cover additional use cases.