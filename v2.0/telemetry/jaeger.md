---
lastmod: 2019-10-11
old_version: true
date: 2019-10-11
notoc: true
linktitle: Jaeger
title: Exporting traces to Jaeger
weight: 100
notoc: true
menu:
  community_v2.0:
    parent: "080 Telemetry and Analytics"
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
[Jaeger](https://www.jaegertracing.io/) is an open source, end-to-end distributed tracing system that allows you to monitor and troubleshoot transactions in complex distributed systems.

The Opencensus exporter allows you export data to Jaeger. Enabling it only requires you to add the `jaeger` exporter in the [opencensus module](/docs/v2.0/telemetry/opencensus/).

The following configuration snippet sends data to your Jaeger:
{{< highlight json >}}
{
  "extra_config":{
    "telemetry/opencensus": {
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
{{< /highlight >}}

- `endpoint` is the URL (including port) where your Jaeger is
- `service_name` the service name registered in Jaeger
- `buffer_max_count` defines the total number of traces that can be buffered in memory


See also the [additional settings](/docs/v2.0/telemetry/opencensus/) of the Opencensus module that can be declared.
