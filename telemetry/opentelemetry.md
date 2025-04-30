---
lastmod: 2024-01-29
date: 2024-01-22
notoc: false
linktitle: OpenTelemetry
title: Telemetry and Monitoring through OpenTelemetry
description: Learn about the telemetry and monitoring capabilities of KrakenD API Gateway using OTEL, enabling real-time visibility and analysis of API performance
weight: 20
images:
- /images/documentation/diagrams/opentelemetry-otlp.mmd.svg
skip_header_image: true
meta:
  since: 2.6
  source: https://github.com/krakend/krakend-otel
  namespace:
  - telemetry/opentelemetry
  scope:
  - service
  log_prefix:
  - "[SERVICE: OpenTelemetry]"
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
---
OpenTelemetry (for short **OTEL**) offers a comprehensive, unified, and **vendor-neutral** approach to collecting and managing telemetry data, providing enhanced observability and deeper insights into application performance and behavior. It's particularly beneficial in complex, distributed, and cloud-native environments.

OpenTelemetry captures detailed, contextual information about the operation of your applications. This includes not only **metrics** but also **tracing data** that shows the full lifecycle of requests as they flow through your systems, providing insights into performance bottlenecks, latency issues, and error diagnostics.

It supports auto-instrumentation and can be integrated seamlessly into cloud-native deployments, making it easier to monitor these dynamic environments.

{{< note title="Stability note on OpenTelemetry" type="note" >}}
KrakenD has traditionally offered part of its telemetry integration through the [OpenCensus integration](/docs/telemetry/opencensus/), which has provided a reliable service for over six years. We are transitioning to the more modern and robust OpenTelemetry framework, and the OpenCensus integration does not receive further updates.

While the underlying protocol specification of OpenTelemetry is stable, you'll find [mixed stability statuses](https://opentelemetry.io/docs/specs/status/) in the components lifecycle. While we cannot predict what changes there will be as the technology evolves, KrakenD will always do its best to maintain compatibility between versions. More information about the underlying exporter can be found [here](https://opentelemetry.io/docs/languages/go/exporters/).
{{< /note >}}


## Collecting metrics and traces
The `telemetry/opentelemetry` component in KrakenD collects the activity generated for the enabled layers and pushes or exposes the data for pulling. There are two ways of publishing metrics:

- **OpenTelemetry protocol (OTLP)** - push
- **Prometheus** - pull

You can use both simultaneously if needed, and even multiple instances of each.

When you add OpenTelemetry in the configuration, you will have [different metrics available](/docs/telemetry/opentelemetry-layers-metrics/).

## Prometheus exporter (pull)
Choose the `prometheus` exporter when you want KrakenD to **expose a new port offering a `/metrics` endpoint**. So, an external Prometheus job can connect to a URL like `http://krakend:9090/metrics` and retrieve all the data.

![Prometheus connecting to KrakenD and fetching metrics](/images/documentation/diagrams/opentelemetry-prometheus.mmd.svg)

[See how to configure Prometheus](/docs/telemetry/prometheus/)

## OTLP exporter (push)
Choose the `otlp` exporter when you want to **push the metrics to a local or remote collector** or directly to a SaaS or storage system that supports native OTLP (there is a [large number of supported providers](/docs/telemetry/#opentelemetry-integrations)). The following diagram represents this idea:

![KrakenD to collector, collector to backend](/images/documentation/diagrams/opentelemetry-otlp.mmd.svg)

The `host` where your collector lives can also point to an external load balancer between KrakenD and multiple collectors if needed:
![KrakenD to load balanced collectors, collectors to backend](/images/documentation/diagrams/opentelemetry-otlp-lb.mmd.svg)


{{< badge >}}Enterprise{{< /badge >}} users can push directly to external storage passing auth credentials using the [`telemetry/opentelemetry-security` component](/docs/enterprise/telemetry/opentelemetry-security/), so the collector is not needed anymore:

![opentelemetry-otlp-auth.mmd diagram](/images/documentation/diagrams/opentelemetry-otlp-auth.mmd.svg)

This strategy saves a lot of time during the setup of KrakenD.


## OpenTelemetry Configuration
To enable OpenTelemetry, you will need a Prometheus or an OTEL Collector (or both) and add the `telemetry/opentelemetry` namespace at the top level of your configuration.


The configuration of the `telemetry/opentelemetry` namespace is very extensive, but the two key entries are:

- `exporters`, defining the different technologies you will use
- `layers`, the amount of data you want to report

The entire configuration is as follows:

{{< schema data="telemetry/opentelemetry.json" >}}

Here's an example with a Grafana Tempo and a Prometheus.

```json
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/v{{< product minor_version >}}/krakend.json",
    "extra_config": {
        "telemetry/opentelemetry": {
            "service_name": "krakend_middle_service",
            "service_version": "commit-sha-ACBDE1234",
            "deploy_env": "production",
            "exporters": {
                "prometheus": [
                    {
                        "name": "my_prometheus",
                        "port": 9092,
                        "listen_ip": "::1",
                        "process_metrics": false,
                        "go_metrics": false
                    }
                ],
                "otlp": [
                    {
                        "name": "local_tempo",
                        "host": "localhost",
                        "port": 4317,
                        "use_http": false
                    }
                ]
            },
            "layers": {
                "global": {
                    "disable_metrics": false,
                    "disable_traces": false,
                    "disable_propagation": false
                },
                "proxy": {
                    "disable_metrics": false,
                    "disable_traces": false
                },
                "backend": {
                    "metrics": {
                        "disable_stage": false,
                        "round_trip": true,
                        "read_payload": true,
                        "detailed_connection": true,
                        "static_attributes": [
                            {
                                "key": "my_metric_attr",
                                "value": "my_middle_metric"
                            }
                        ]
                    },
                    "traces": {
                        "disable_stage": false,
                        "round_trip": true,
                        "read_payload": true,
                        "detailed_connection": true,
                        "static_attributes": [
                            {
                                "key": "my_metric_attr",
                                "value": "my_middle_metric"
                            }
                        ]
                    }
                }
            },
            "skip_paths": [
                "/foo/{bar}"
            ]
        }
    }
}
```
## Examples of integrations
- Push metrics to [InfluxDB](/docs/telemetry/influxdb/)
- Pull metrics from [Prometheus](/docs/telemetry/prometheus/)
- Push metrics to [Datadog](/docs/telemetry/datadog/)
- Push metrics to [Zipkin](/docs/telemetry/zipkin/)
- Push metrics to [Jaeger](/docs/telemetry/jaeger/)
- Push metrics to [Azure Monitor](/docs/telemetry/azure/)