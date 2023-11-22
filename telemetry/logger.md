---
lastmod: 2022-10-24
date: 2019-09-15
notoc: true
linktitle: Logging through OpenCensus
title: Exporting to the logger with OpenCensus
description: Utilize an alternative Logger telemetry integration based on OpenCensus to monitor and analyze the API Gateway in KrakenD.
weight: 150
notoc: true
aliases: ["/docs/logging-metrics-tracing/logger/"]
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
{{< note title="Use standard logging instead" type="info" >}}
Unless you have a compelling reason to use this logger based on OpenCensus, you should use the [standard KrakenD Logging](/docs/logging/) and not this component. This component will be deprecated in the future.
{{< /note >}}

Opencensus can also export data to the system logger as other exporters. If you use this component, **do not not enable standard logging**.

To enable OpenCensus logging, it only requires you to add the `logger` exporter in the [opencensus module](/docs/telemetry/opencensus/).

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
