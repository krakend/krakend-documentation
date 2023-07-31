---
lastmod: 2022-10-24
old_version: true
date: 2020-07-24
notoc: true
linktitle: Datadog
title: Exporting traces to Datadog
weight: 90
since: 1.2
source: https://github.com/krakend/krakend-opencensus
notoc: true
images: ["/images/documentation/datadog-screenshot.png"]
menu:
  community_v2.1:
    parent: "080 Telemetry and Analytics"
meta:
  since: 1.2
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
[Datadog](https://www.datadoghq.com/) is a cloud monitoring and security platform for developers, IT operations teams, and businesses.

## Datadog configuration
The Opencensus exporter allows you to export data to Datadog. Enabling it only requires you to add the `datadog` exporter in the [opencensus module](/docs/v2.1/telemetry/opencensus/).

The following configuration snippet sends data to your Datadog:
```json
{
      "version": 3,
      "extra_config": {
        "telemetry/opencensus": {
          "sample_rate": 100,
          "reporting_period": 0,
          "exporters": {
            "datadog": {
              "tags": [
                "gw"
              ],
              "global_tags": {
                "env": "prod"
              },
              "disable_count_per_buckets": true,
              "trace_address": "localhost:8126",
              "stats_address": "localhost:8125",
              "namespace": "krakend",
              "service": "gateway"
            }
          }
        }
      }
}
```
As with all [OpenCensus exporters](/docs/v2.1/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.1" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `datadog` entry with the following properties:

{{< schema version="v2.1" data="telemetry/opencensus.json" property="exporters" filter="datadog" >}}

## B3 propagation
The Opencensus module uses B3 style propagation headers, while the rest of your services might be using datadog-specific propagation headers. If this difference is actual, krakend traces will show up in Datadog but they won't be connected to the frontend and backend traces.

The `ddtrace-run` adds an option to support B3 style propagation using the environment variables `DD_TRACE_PROPAGATION_STYLE_EXTRACT` and `DD_TRACE_PROPAGATION_STYLE_INJECT`. Use these variables to have your traces perfectly aligned.

For more information see [its configuration](https://ddtrace.readthedocs.io/en/stable/configuration.html).