---
lastmod: 2024-02-27
date: 2019-09-15
notoc: true
linktitle: Zipkin
title: Zipkin Telemetry Integration
description: Integrate Zipkin telemetry to monitor and trace KrakenD API Gateway requests efficiently for enhanced performance
weight: 100
notoc: true
aliases: ["/docs/logging-metrics-tracing/zipkin/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
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
[Zipkin](https://zipkin.io/) is a distributed tracing system. It helps gather timing data needed to troubleshoot latency problems in service architectures.

The Opencensus exporter allows you export data to Zipkin. Enabling it only requires you to add the `zipkin` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your Zipkin:
```json
{
  "version": 3,
  "extra_config": {
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 0,
      "exporters": {
        "zipkin": {
          "collector_url": "http://192.168.99.100:9411/api/v2/spans",
          "service_name": "krakend"
        }
      }
    }
  }
}
```

As with all [OpenCensus exporters](/docs/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `zipkin` entry with the following properties:

{{< schema data="telemetry/opencensus.json" property="exporters" filter="zipkin" >}}
