---
lastmod: 2021-05-02
old_version: true
date: 2018-10-29
linktitle: Proxy rate limit
title: Rate-limiting backends
weight: 10
menu:
  community_v2.2:
    parent: "050 Backends Configuration"
notoc: true
meta:
  since: false
  source: https://github.com/krakendio/krakend-ratelimit
  namespace:
  - qos/ratelimit/proxy
  scope:
  - backend
---

No matter what is the amount of activity the users are generating at the router level, you might want to restrict the connections KrakenD makes to your backends. Configuration is similar to the router's one, but it's declared directly in the `backend` section instead of the `endpoint`.

This parameter is defined at the `krakend.json` configuration file as follows:
{{< highlight json "hl_lines=8-13">}}
    {
      "endpoint": "/products/{cat_id}",
      "backend": [
      {
          "host": ["http://some.api.com/"],
          "url_pattern": "/catalog/category/{cat_id}.rss",
          "encoding": "rss",
          "extra_config": {
              "qos/ratelimit/proxy": {
                  "max_rate": 0.5,
                  "capacity": 1
              }
          }
      }
{{< /highlight >}}

These are two parameters you can set:

{{< schema version="v2.2" data="qos/ratelimit/proxy.json" >}}

## Comparison of `max_rate` vs `client_max_rate`
The `max_rate` (available both in router and proxy layers) is an absolute number where you have the exact control over how much traffic you are allowing to hit the backend or endpoint. In an eventual DDoS, the `max_rate` can help in a way since it won't accept more traffic than allowed. But on the other hand a single host could abuse the system taking a big percentage of that quota.

The `client_max_rate` is a limit per client and it won't help you if you just want to control the total traffic, as the total traffic supported by the backend or endpoint depends on the number of different requesting clients. A DDoS will then happily pass through, but on the other hand, you can keep any particular abuser limited to its quota.

Depending on your use case you will need to decide if you use one, the other, the two, or none of them.
