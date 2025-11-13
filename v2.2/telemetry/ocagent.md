---
lastmod: 2022-10-24
old_version: true
date: 2020-11-16
canonical: "/docs/v2.5/telemetry/ocagent/"
notoc: true
linktitle: OpenCensus Agent
title: Exporting metrics, logs, and events to the OpenCensus Agent
weight: 140
menu:
  community_v2.2:
    parent: "080 Telemetry and Analytics"
meta:
  since: v1.1
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---

The `ocagent` exporter sends OpenCensus Stats and Traces to the OpenCensus Agent, instead of pushing data to backends’ exporters.

For instance, you can enable ocagent to upload data to the OpenCensus Agent, and from there, the agent is simply scraped by a Prometheus.

You can integrate the OpenCensus Agent with Azure Monitor, Jaeger, or Prometheus to name a few examples.

Enabling it only requires you to add the `ocagent` exporter in the [opencensus module](/docs/v2.2/telemetry/opencensus/).

The following configuration snippet sends the data:

```json
{
  "extra_config": {
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 0,
      "exporters": {
       "ocagent": {
          "address": "collector",
          "service_name": "krakend",
          "reconnection": "2s",
           "insecure": false,
          "enable_compression": true,
          "headers": {
            "header1": "value1"
          }
        }
      }
    }
}
```


As with all [OpenCensus exporters](/docs/v2.2/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.2" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `ocagent` entry with the following properties:

{{< schema version="v2.2" data="telemetry/opencensus.json" property="exporters" filter="ocagent" >}}
