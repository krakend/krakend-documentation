---
lastmod: 2024-01-22
date: 2019-09-15
notoc: true
linktitle: OpenCensus (frozen)
title: OpenCensus Telemetry Integration
description: The OpenCensus telemetry integration is now development-frozen. All efforts are now put into OpenTelemetry.
weight: 60
aliases: ["/docs/logging-metrics-tracing/opencensus/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: 0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
{{< note title="OpenCensus is no longer mantained" type="error" >}}
KrakenD has traditionally offered its telemetry integration through this **OpenCensus** component, which has provided reliable service for over six years, but now is transitioning to the more modern and robust [OpenTelemetry](/docs/telemetry/opentelemetry/) framework.

As a result of a change in the industry, the OpenCensus integration is no longer mantained, and all efforts are focused on [OpenTelemetry](/docs/telemetry/opentelemetry/).
{{< /note >}}

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

By default, all exporters use a `sample_rate=0`, meaning that **they won't report anything**, but this can be changed by specifying the configuration another percentage, like 100% of the activity:

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

{{< schema data="telemetry/opencensus.json" norecurse="exporters">}}



## Transition from OpenCensus
If you have been using telemetry on KrakenD for the past six years (before KrakenD 2.6), you were using the [OpenCensus](/docs/telemetry/opencensus/) exporters, which have worked like a Swiss clock.

While OpenCensus has been working very well, it has merged with OpenTracing to form [OpenTelemetry](https://opentelemetry.io/), which serves as the next major version of OpenCensus and OpenTracing.

While our OpenCensus integration will keep functioning on KrakenD for now, it won't receive additional updates (netiher security fixes), so we recommend replacing it with OpenTelemetry.