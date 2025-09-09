---
lastmod: 2019-09-15
old_version: true
date: 2019-09-15
notoc: true
linktitle: Opencensus
title: Sending out logs, metrics, and traces
weight: 60
menu:
  community_v1.3:
    parent: "080 Telemetry"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
The Opencensus exporter is a single component that allows you to **export data to multiple providers**, both open source and privative.

You will be interested in Opencensus when you want to see data in one of its supported `exporters`. For instance, you might want to send metrics to Prometheus. That would be as easy as adding this snippet in the **root level** of your `krakend.json` file:

    {
        "version": 2,
        "extra_config": {
            "github_com/devopsfaith/krakend-opencensus": {
                "exporters": {
                    "prometheus": {
                        "port": 9091
                        "namespace": "krakend"
                    }
                }
            }
        }
    }

## Configuration

The Opencensus only needs an exporter to work, although multiple exporters can be added in the same configuration. Every exporter has its own configuration and this described in its own section.

By default, **all exporters sample the 100% of the requests received every second**, but this can be changed by specifying more configuration:

    "github_com/devopsfaith/krakend-opencensus": {
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

- `sample_rate` is the percentage of sampled requests. A value of `100` means that all requests are exported (100%). If you are processing a huge amount of traffic you might want to sample only a part of what's going on.
- `reporting_period` is the number of **seconds** passing between reports
- `exporters` is a key-value with all the exporters you want to use. See each exporter configuration for the underlying keys.
- `enabled_layers` let you specify what data you want to export. All layers are enabled by default:
  - Use `backend` to report the activity between KrakenD and your services
  - Use `router` to report the activity between end-users and KrakenD
  - Use `pipe` to report the activity at the beginning of the proxy layer. It gives a more detailed view of the internals of the pipe between end-users and KrakenD.
