---
lastmod: 2022-10-24
old_version: true
date: 2019-09-15
canonical: /docs/v2.5/telemetry/logger/
notoc: true
linktitle: Logging through OpenCensus
title: Exporting to the logger with OpenCensus
weight: 150
notoc: true
menu:
  community_v2.2:
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
Opencensus can export data to the system logger as another exporter. This **is not** the [standard KrakenD Logging](/docs/v2.2/logging/), and you should not enable both.

Enabling it only requires you to add the `logger` exporter in the [opencensus module](/docs/v2.2/telemetry/opencensus/).

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

As with all [OpenCensus exporters](/docs/v2.2/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.2" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `logger` entry with the following properties:

{{< schema version="v2.2" data="telemetry/opencensus.json" property="exporters" filter="logger" >}}
