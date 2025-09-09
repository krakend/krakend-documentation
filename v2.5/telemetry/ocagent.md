---
lastmod: 2022-10-24
old_version: true
aliases: ["/docs/telemetry/ocagent/"]
date: 2020-11-16
notoc: true
linktitle: OpenCensus Agent
title: OpenCensus Agent
description: The ocagent exporter sends OpenCensus Stats and Traces to the OpenCensus Agent, instead of pushing data to backends’ exporters
weight: 70
notoc: true
menu:
  community_v2.5:
    parent: "160 Monitoring, Logs, and Analytics"
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
{{< note title="OpenCensus is no longer mantained" type="error" >}}
KrakenD has traditionally offered its telemetry integration through this **OpenCensus** component, which has provided reliable service for over six years, but  now has transitioned to the more modern and robust [OpenTelemetry](/docs/telemetry/opentelemetry/) framework.

As a result of a change in the industry, the OpenCensus integration is no longer mantained, we recommend you to migrate to [OpenTelemetry](/docs/telemetry/opentelemetry/).
{{< /note >}}

The `ocagent` exporter sends OpenCensus Stats and Traces to the OpenCensus Agent, instead of pushing data to backends’ exporters.

For instance, you can enable ocagent to upload data to the OpenCensus Agent, and from there, the agent is simply scraped by a Prometheus.

You can integrate the OpenCensus Agent with Azure Monitor, Jaeger, or Prometheus to name a few examples.

Enabling it only requires you to add the `ocagent` exporter in the [opencensus module](/docs/v2.5/telemetry/opencensus/).

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


As with all [OpenCensus exporters](/docs/v2.5/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.5" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `ocagent` entry with the following properties:

{{< schema version="v2.5" data="telemetry/opencensus.json" property="exporters" filter="ocagent" >}}
