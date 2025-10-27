---
lastmod: 2024-05-15
old_version: true
date: 2020-07-24
notoc: true
linktitle: Datadog
title: Datadog Telemetry Integration
description: Integrate Datadog telemetry with KrakenD API Gateway for advanced monitoring, visualization, and analysis of your API ecosystem
weight: 90
since: 1.2
source: https://github.com/krakend/krakend-opencensus
notoc: true
images: ["/images/documentation/datadog-screenshot.png"]
menu:
  community_v2.11:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: v1.2
  source: https://github.com/krakend/krakend-otel
  namespace:
  - telemetry/opentelemetry
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
[Datadog](https://www.datadoghq.com/) is a cloud monitoring and security platform for developers, IT operations teams, and businesses.

The [OpenTelemetry integration](/docs/v2.11/telemetry/opentelemetry/) allows you to send **metrics and traces** to Datadog using their collector.

## Datadog configuration
Datadog uses the standard OTLP exporter, here is a configuration example:

```json
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/krakend.json",
    "host": [
        "http://localhost:8080"
    ],
    "debug_endpoint": true,
    "echo_endpoint": true,
    "extra_config": {
        "telemetry/opentelemetry": {
            "exporters": {
                "otlp": [
                    {
                        "use_http": false,
                        "port": 4317,
                        "host": "ddagent",
                        "name": "my_dd_exporter",
                        "disable_metrics": false,
                        "disable_traces": false
                    }
                ]
            },
            "trace_sample_rate": 1,
            "service_name": "krakend_dd_telemetry",
            "metric_reporting_period": 1
        }
    }
}
```

The important part of the configuration is the `otlp` exporter, which accepts the following fields:

{{< schema version="v2.11" data="telemetry/opentelemetry.json" property="exporters" filter="otlp">}}

In addition, you can configure how the `layers` behave ([see all options](/docs/v2.11/telemetry/opentelemetry/#layers)).


## Datadog agent
You must set your Datadog API key in the agent. The exporter communicates with the agent and is the agent the one reporting to Datadog.

Here's an example of how to run the Datadog agent together with KrakenD in a docker compose file:

```yml
version: '3'
services:
  krakend:
    image: {{< product image >}}:2.11
    volumes:
      - "./:/etc/krakend"
    command: ["run", "-c", "krakend.json"]
    ports:
      - "8080:8080"
  datadog:
    image: gcr.io/datadoghq/agent:7
    pid: host
    environment:
     - DD_API_KEY=XXXXXXXXXXXXXXX
     - DD_OTLP_CONFIG_RECEIVER_PROTOCOLS_GRPC_ENDPOINT=0.0.0.0:4317
     - DD_OTLP_CONFIG_RECEIVER_PROTOCOLS_HTTP_ENDPOINT=0.0.0.0:4318
     - DD_SITE=datadoghq.com
    volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /proc/:/host/proc/:ro
     - /sys/fs/cgroup:/host/sys/fs/cgroup:ro
```

Notice that we are naming the service `ddagent` in Docker compose, and this matches our `host` field in the configuration.

## Migrating from OpenCensus
Prior to v2.6, telemetry sent to Datadog used the OpenCensus exporter. Enabling required adding the `datadog` exporter in the [opencensus module](/docs/v2.11/telemetry/opencensus/), and the configurations looked like this:
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
You can migrate to OpenTelemetry doing the following changes:

- Rename `telemetry/opencensus` to `telemetry/opentelemetry`.
- `sample_rate` -> Delete this field
- `reporting_period` -> Rename to `metric_reporting_period`
- `datadog` -> Rename to `otlp`, and add an array surrounding the object, so it becomes `"otlp": [{...}]`
- `namespace` -> Rename to `name`
- `tag_host`, `tag_path`,`tag_method`,`tag_statuscode` -> Delete them
