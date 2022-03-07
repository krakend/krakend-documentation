---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Opencensus
title: Sending out logs, metrics, and traces
weight: 60
aliases: ["/docs/logging-metrics-tracing/opencensus/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/devopsfaith/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
The Opencensus exporter is a single component that allows you to **export data to multiple providers**, both open source and privative.

You will be interested in Opencensus when you want to see data in one of its supported `exporters`. For instance, you might want to send metrics to Prometheus. That would be as easy as adding this snippet in the **root level** of your `krakend.json` file:

{{< highlight json >}}
{
    "version": 3,
    "extra_config": {
        "telemetry/opencensus": {
            "exporters": {
                "prometheus": {
                    "port": 9091,
                    "namespace": "krakend"
                }
            }
        }
    }
}
{{< /highlight >}}

## Configuration

The Opencensus needs at least an exporter to work, although multiple exporters can be added in the same configuration. Notice that adding several exporters that push metrics out to multiple systems will affect the performance of the server.

Every exporter has its own configuration and it's described in its section.

By default, **all exporters sample the 100% of the requests received every second**, but this can be changed by specifying more configuration:

{{< highlight json >}}
{
    "version": 3,
    "extra_config": {
        "telemetry/opencensus": {
            "sample_rate": 100,
            "reporting_period": 1,
            "enabled_layers": {
                "backend": true,
                "router": true,
                "pipe": true
            },
            "exporters": {
                "prometheus": {
                    "port": 9091
                }
            }
        }
    }
}
{{< /highlight >}}



- `sample_rate` is the percentage of sampled requests. A value of `100` means that all requests are exported (100%). If you are processing a huge amount of traffic you might want to sample only a part of what's going on.
- `reporting_period` is the number of **seconds** passing between reports
- `exporters` is a key-value with all the exporters you want to use. See each exporter configuration for the underlying keys.
- `enabled_layers` let you specify what data you want to export. All layers are enabled by default:
  - Use `backend` to report the activity between KrakenD and your services
  - Use `router` to report the activity between end-users and KrakenD
  - Use `pipe` to report the activity at the beginning of the proxy layer. It gives a more detailed view of the internals of the pipe between end-users and KrakenD.
