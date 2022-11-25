---
lastmod: 2022-10-24
date: 2019-10-11
notoc: true
linktitle: Jaeger
title: Exporting traces to Jaeger
weight: 100
notoc: true
aliases: ["/docs/logging-metrics-tracing/jaeger/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/krakendio/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
[Jaeger](https://www.jaegertracing.io/) is an open source, end-to-end distributed tracing system that allows you to monitor and troubleshoot transactions in complex distributed systems.

The Opencensus exporter allows you export data to Jaeger. Enabling it only requires you to add the `jaeger` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your Jaeger:
```json
{
  "extra_config":{
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 0,
      "exporters": {
        "jaeger": {
          "endpoint": "http://192.168.99.100:14268/api/traces",
          "service_name":"krakend",
          "buffer_max_count": 1000
        }
      }
    }
  }
}
```

As with all [OpenCensus exporters](/docs/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `jaeger` entry with the following properties:

{{< schema data="telemetry/opencensus.json" property="exporters" filter="jaeger" >}}
