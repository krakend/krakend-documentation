---
lastmod: 2022-10-24
old_version: true
date: 2019-09-15
notoc: true
linktitle: Opencensus
title: Sending out logs, metrics, and traces
weight: 20
menu:
  community_v2.2:
    parent: "080 Telemetry and Analytics"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
The Opencensus exporter is a single component that allows you to **export data to multiple providers**, both open source and privative.

You will be interested in Opencensus when you want to see data in one of its supported `exporters`. For instance, you might want to send metrics to Prometheus. That would be as easy as adding this snippet in the **root level** of your `krakend.json` file:

```json
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
```

## Configuration

The Opencensus needs at least an exporter to work, although multiple exporters can be added in the same configuration. Notice that adding several exporters that push metrics out to multiple systems will affect the performance of the server.

Every exporter has its own configuration and it's described in its section.

By default, **all exporters use a sample_rate=0**, meaning that they won't report anything, but this can be changed by specifying the de configuration:

```json
{
    "version": 3,
    "extra_config": {
        "telemetry/opencensus": {
            "sample_rate": 100,
            "reporting_period": 0,
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
```

{{< schema version="v2.2" data="telemetry/opencensus.json" norecurse="exporters">}}
