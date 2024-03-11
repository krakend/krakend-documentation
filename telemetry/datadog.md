---
lastmod: 2024-02-27
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
aliases: ["/docs/logging-metrics-tracing/datadog/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
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

The [OpenTelemetry integration](/docs/telemetry/opentelemetry/) allows you to send **metrics and traces** to Datadog using a Collector.
## Datadog configuration
Datadog uses the standard OTLP exporter, here is a configuration example:

```json
{
  "version": 3,
  "extra_config": {
        "telemetry/opentelemetry": {
            "service_name": "krakend_service",
            "metric_reporting_period": 1,
            "@comment": "Report 20% of traces",
            "trace_sample_rate": 0.2,
            "exporters": {
                "otlp": [
                    {
                        "name": "my_datadog_agent",
                        "host": "ddagent",
                        "port": 8126,
                        "use_http": false
                    }
                ]
            }
        }
    }
}
```

The important part of the configuration is the `otlp` exporter, which accepts the following fields:

{{< schema data="telemetry/opentelemetry.json" property="exporters" filter="otlp">}}

In addition, you can configure how the `layers` behave ([see all options](/docs/telemetry/opentelemetry/#layers)).

## Datadog agent
You must set your Datadog API key in the agent. The exporter communicates with the agent and is the agent the one reporting to Datadog.

Here's an example of how to run the Datadog agent together with KrakenD in a docker-compose file:

```yml
krakend:
  image: {{< product image >}}:{{< product latest_version >}}
ddagent:
  image: gcr.io/datadoghq/agent:latest
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - /proc/:/host/proc/:ro
    - /sys/fs/cgroup/:/host/sys/fs/cgroup:ro
  ports:
    - 8126:8126/tcp
    - 8125:8125/udp
  environment:
    # Replace with your API key:
    - DD_API_KEY=<API-KEY>
    - DD_APM_ENABLED=true
    # Replace with your site: https://docs.datadoghq.com/getting_started/site/
    - DD_SITE=<DATADOG-SITE>
    - DD_APM_NON_LOCAL_TRAFFIC=true
```

And the configuration would contain:

```json
{ "datadog":
  {
    "@comment": "Rest of the necessary fields intentionally omitted",
    "trace_address": "ddagent:8126",
    "stats_address": "ddagent:8125"
  }
}
```

Notice that we are naming the service `ddagent` in Docker compose, and this is what we have added in the address.

## Migrating from OpenCensus
Prior to v2.6, telemetry sent to Datadog used the OpenCensus exporter. Enabling required adding the `datadog` exporter in the [opencensus module](/docs/telemetry/opencensus/), and the configurations looked like this:
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
