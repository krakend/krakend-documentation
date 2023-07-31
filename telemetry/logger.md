---
lastmod: 2022-10-24
date: 2019-09-15
notoc: true
linktitle: Logging through OpenCensus
title: Exporting to the logger with OpenCensus
weight: 150
notoc: true
aliases: ["/docs/logging-metrics-tracing/logger/"]
menu:
  community_current:
    parent: "090 Logging"
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
Opencensus can export data to the system logger as another exporter. This **is not** the [standard KrakenD Logging](/docs/logging/), and you should not enable both.

Enabling it only requires you to add the `logger` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet enables the logger:
```json
{
  "extra_config":{
    "telemetry/opencensus": {
        "sample_rate": 100,
        "reporting_period": 0,
        "exporters": {
          "logger": {
              "stats": true,
              "spans": true
          }
        }
    }
}
```

As with all [OpenCensus exporters](/docs/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `logger` entry with the following properties:

{{< schema data="telemetry/opencensus.json" property="exporters" filter="logger" >}}
