---
lastmod: 2021-09-01
date: 2020-07-24
notoc: true
linktitle: Datadog
title: Exporting traces to Datadog
weight: 90
since: 1.2
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
images: ["/images/documentation/datadog-screenshot.png"]
aliases: ["/docs/logging-metrics-tracing/datadog/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 1.2
  source: https://github.com/devopsfaith/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
[Datadog](https://www.datadoghq.com/) is a monitoring and security platform for developers, IT operations teams and business in the cloud.

## Datadog configuration
The Opencensus exporter allows you export data to Datadog. Enabling it only requires you to add the `datadog` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your Datadog:
{{< highlight json >}}
{
      "extra_config": {
        "telemetry/opencensus": {
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
{{< /highlight  >}}
- `namespace`(*string*) the namespace to which metric keys are appended.
- `tags` (*list*) specifies a set of global tags to attach to each metric
- `global_tags` (*object*) GlobalTags holds a set of tags (key/value) that will automatically be applied to all exported spans.
- `service` (*string*) Service specifies the service name used for tracing
- `trace_address` (*string*) TraceAddr specifies the `host[:port]` address of the Datadog Trace Agent. It defaults to localhost:8126.
- `stats_address` (*string*) StatsAddr specifies the `host[:port]` address for DogStatsD. It defaults to localhost:8125. To enable ingestion using [Unix Domain Socket (UDS)](https://docs.datadoghq.com/developers/dogstatsd/unix_socket/?tab=kubernetes) mount your UDS path and reference it in the `stats_address` using a path like `unix:///var/run/datadog/dsd.socket`.
- `disable_count_per_buckets` (*bool*) Specifies whether to emit count_per_bucket metrics


See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.
