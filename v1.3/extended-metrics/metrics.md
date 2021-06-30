---
lastmod: 2021-05-02
old_version: true
date: 2018-11-05
linktitle: Metrics and /__stats
title: Extended metrics and the `/__stats` endpoint
weight: 10
menu:
  community_v1.3:
    parent: "100 Extended Metrics"
notoc: true
meta:
  since: 0.4
  source: https://github.com/devopsfaith/krakend-metrics
  namespace:
  - github_com/devopsfaith/krakend-metrics
  scope:
  - service
---

The metrics *middleware* collects **extended metrics** and exposes them in the `/__stats/` endpoint. The endpoint runs in a different port than the API, and contains a lot of metrics.

Through the extended metrics you can create new tools or integrate with existing ones. For instance, combining the metrics with the InfluxDB extended metrics you can have a [Grafana dashboard](/docs/v1.3/extended-metrics/grafana/).

## Configuration

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
    }
  }
}
{{< /highlight >}}

The options of the *middleware* are:

- `collection_time`: The time window to collect metrics. Defaults to 60 seconds.
- `proxy_disabled`: Skip any metrics happening in the proxy layer (traffic against your backends)
- `router_disabled`:  Skip any metrics happening in the router layer (activity in KrakenD endpoints)
- `backend_disabled`: Skip any metrics happening in the backend layer.
- `endpoint_disabled`: Do not publish the `/__stats/` endpoint. Metrics won't be accessible via the endpoint but still collected.
- `listen_address`: Change the listening address where the metrics endpoint is exposed. It defaults to :8090.

The structure of the metrics looks like this (truncated):

{{< terminal title="Sample of /__stats endpoint" >}}
curl http://localhost:8090/__stats
{
  "cmdline": [
    "/usr/bin/krakend",
    "run",
    "-c",
    "/etc/krakend/krakend.json",
    "-d"
  ],
  "krakend.router.connected": 0,
  "krakend.router.connected-gauge": 0,
  "krakend.router.connected-total": 0,
  "krakend.router.disconnected": 0,
  "krakend.router.disconnected-gauge": 0,
  "krakend.router.disconnected-total": 0,
  "krakend.service.debug.GCStats.LastGC": 1605724147216402400,
  "krakend.service.debug.GCStats.NumGC": 102,
  "krakend.service.debug.GCStats.Pause.50-percentile": 0,
  "krakend.service.debug.GCStats.Pause.75-percentile": 0,
  "krakend.service.debug.GCStats.Pause.95-percentile": 0,
  "krakend.service.debug.GCStats.Pause.99-percentile": 0,
  "krakend.service.debug.GCStats.Pause.999-percentile": 0,
  "krakend.service.debug.GCStats.Pause.count": 0,
  "krakend.service.debug.GCStats.Pause.max": 0,
  "krakend.service.debug.GCStats.Pause.mean": 0,
  "krakend.service.debug.GCStats.Pause.min": 0,
  "krakend.service.debug.GCStats.Pause.std-dev": 0,
    ...
  }
}
{{< /terminal >}}
