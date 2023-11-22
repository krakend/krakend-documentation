---
lastmod: 2022-10-24
date: 2020-11-16
notoc: true
linktitle: OpenCensus Agent
title: OpenCensus Agent Telemetry Integration
description: The ocagent exporter sends OpenCensus Stats and Traces to the OpenCensus Agent, instead of pushing data to backends’ exporters
weight: 140
notoc: true
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: 1.1
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

Enabling it only requires you to add the `ocagent` exporter in the [opencensus module](/docs/telemetry/opencensus/).

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


As with all [OpenCensus exporters](/docs/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `ocagent` entry with the following properties:

{{< schema data="telemetry/opencensus.json" property="exporters" filter="ocagent" >}}
