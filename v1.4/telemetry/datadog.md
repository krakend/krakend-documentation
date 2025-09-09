---
lastmod: 2021-09-01
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
  community_v1.4:
    parent: "080 Telemetry"
meta:
  since: v1.2
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
[Datadog](https://www.datadoghq.com/) is a monitoring and security platform for developers, IT operations teams and business in the cloud.

## Datadog configuration
The Opencensus exporter allows you export data to Datadog. Enabling it only requires you to add the `datadog` exporter in the [opencensus module](/docs/v1.4/telemetry/opencensus/).

The following configuration snippet sends data to your Datadog:

      "extra_config": {
        "github_com/devopsfaith/krakend-opencensus": {
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

- `tags` (*list*) specifies a set of global tags to attach to each metric
- `global_tags` (*object*) GlobalTags holds a set of tags (key/value) that will automatically be applied to all exported spans.
- `service` (*string*) Service specifies the service name used for tracing
- `trace_address` (*string*) TraceAddr specifies the `host[:port]` address of the Datadog Trace Agent. It defaults to localhost:8126.
- `stats_address` (*string*) StatsAddr specifies the `host[:port]` address for DogStatsD. It defaults to localhost:8125. To enable ingestion using [Unix Domain Socket (UDS)](https://docs.datadoghq.com/developers/dogstatsd/unix_socket/?tab=kubernetes) mount your UDS path and reference it in the `stats_address` using a path like `unix:///var/run/datadog/dsd.socket`.
- `disable_count_per_buckets` (*bool*) Specifies whether to emit count_per_bucket metrics


See also the [additional settings](/docs/v1.4/telemetry/opencensus/) of the Opencensus module that can be declared.
