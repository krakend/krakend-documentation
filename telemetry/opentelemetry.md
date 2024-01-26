---
lastmod: 2024-01-22
date: 2024-01-22
notoc: false
linktitle: OpenTelemetry
title: Telemetry and Monitoring through OpenTelemetry
description: Learn about the telemetry and monitoring capabilities of KrakenD API Gateway using OTEL, enabling real-time visibility and analysis of API performance
weight: 20
#images:
#- /images/documentation/available-exporters.png
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
---
OpenTelemetry (for short **OTEL**) offers a comprehensive, unified, and **vendor-neutral** approach to collecting and managing telemetry data, providing enhanced observability and deeper insights into application performance and behavior. It's particularly beneficial in complex, distributed, and cloud-native environments.

OpenTelemetry captures detailed, contextual information about the operation of your applications. This includes not only **metrics** but also **tracing data** that shows the full lifecycle of requests as they flow through your systems, providing insights into performance bottlenecks, latency issues, and error diagnostics.

It supports auto-instrumentation and can be integrated seamlessly into cloud-native deployments, making it easier to monitor these dynamic environments.

## Collecting metrics and traces
The `telemetry/opentelemetry` component in KrakenD collects the activity generated for the enabled layers, and pushes or exposes the data. There are two different ways of publishing metrics: With the OpenTelemetry protocol (exporter = `otlp`), or with Prometheus (exporter = `prometheus`).

Choose OTLP when you want to **push the metrics to a local or remote collector**. The following diagram represents this idea:

![KrakenD to collector, collector to backend](/images/documentation/diagrams/opentelemetry-otlp.mmd.png)

You can also have an external load balancer between KrakenD and multiple collectors if needed.

Choose the Prometheus exporter when you want KrakenD to **expose a new port offering a `/metrics` endpoint**. So an external Prometheus job can connect to an URL like `http://krakendhost:9090/metrics` and retrieve all the data.

![Prometheus connecting to KrakenD and fetching metrics](/images/documentation/diagrams/opentelemetry-prometheus.mmd.png)

When using OTLP, you can push data to a [large number of providers](/docs/telemetry/#opentelemetry-integrations).

## OTEL Configuration
To enable OpenTelemetry, you will need a Prometheus or an OTEL Collector (or both) and add the `telemetry/opentelemetry` namespace at the top level of your configuration.

Inside the namespace, you will need to add all the `exporters` that you are willing to use. KrakenD will send data to all the declared exporters and layers by default. Exporters define **where** you will have the metrics.

In addition, you can set the `layers` of metrics you want to have. The layers contain the traces and metrics for a subset of the execution flow. These are the layers you can use:

![Diagram showing global, proxy, and backend sequence](/images/documentation/diagrams/opentelemetry-layers.mmd.png)

- `global`: The global layer contains everything that KrakenD saw in and out.
- `proxy`: When the API Gateway deals with an internal request after havin applied  one of the exposed endpoints and includes spawning the required requests to the backends, as well as the manipulation and aggregation at the endpoint level before and after the requests are performed.
- `backend`: the part monitoring a single backend request and response with any additional components in its level. This contains mostly the timings between KrakenD and your service.

Is important to notice that `backend` is a subset of `proxy`, and `proxy` a subset of `global`.

{{< note title="CPU and memory consumption" type="warning" >}}
The more layers and detail you add, the more resources KrakenD will consume, and more storage you will need to save the metrics and traces. Choose carefully a good balance between observability power and resource consumption.
{{< /note >}}

The configuration of the `telemetry/opentelemetry` namespace accepts the following fields:

{{< schema data="telemetry/opentelemetry.json" >}}



```json
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/v2.6/krakend.json",
    "extra_config": {
        "telemetry/opentelemetry": {
            "service_name": "krakend_middle_service",
            "exporters": {
                "prometheus": [
                    {
                        "name": "remote_prometheus",
                        "port": 9092,
                        "listen_ip": "::1",
                        "process_metrics": false,
                        "go_metrics": false
                    },
                    {
                        "name": "local_prometheus",
                        "port": 9093,
                        "process_metrics": true,
                        "go_metrics": true
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
                ""
            ]
        }
    }
}
```
