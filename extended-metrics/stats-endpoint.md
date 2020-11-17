---
lastmod: 2018-11-05
date: 2018-11-05
linktitle: Stats endpoint
title: The stats endpoint
weight: 20
notoc: true
source: https://github.com/devopsfaith/krakend-metrics
aliases:
- /docs/logging-metrics-tracing/metrics/

menu:
  documentation:
    parent: extended-metrics
---
The metrics *middleware* offers a new service in a different port and exposes a `/__stats/` endpoint with the collection of all the KrakenD metrics:

    http://localhost:8090/__stats/

Through the extended metrics you can create new tools or integrate with existing ones. A good example of this is [importing a Grafana dashboard](/docs/extended-metrics/grafana) with all kind of KrakenD metrics.

## Enabling metrics

In order to add metrics to your KrakenD installation add the `github_com/devopsfaith/krakend-metrics` namespace under `extra_config` in the root of your configuration file, e.g.:

{{< highlight go "hl_lines=3-11" >}}
{
  "version": 2,
  "extra_config": {
    "github_com/devopsfaith/krakend-metrics": {
      "collection_time": "60s",
      "proxy_disabled": false,
      "router_disabled": false,
      "backend_disabled": false,
      "endpoint_disabled": false,
      "listen_address": ":8090"
    },
    ...
  }
{{< /highlight >}}

The options of the *middleware* are:

- `collection_time`: The time window to collect metrics. Defaults to 60 seconds.
- `proxy_disabled`: Skip any metrics happening in the proxy layer (traffic against your backends)
- `router_disabled`:  Skip any metrics happening in the router layer (activity in KrakenD endpoints)
- `backend_disabled`: Skip any metrics happening in the backend layer.
- `endpoint_disabled`: Do not publish the `/__stats/` endpoint. Metrics won't be accessible via the endpoint but still collected.
- `listen_address`: Change the listening address where the metrics endpoint is exposed. It defaults to :8090.
