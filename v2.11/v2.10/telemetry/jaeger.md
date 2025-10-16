---
lastmod: 2023-02-03
old_version: true
old_version: true
date: 2019-10-11
notoc: true
linktitle: Jaeger
title: Jaeger Telemetry Integration - KrakenD API Gateway
description: Integrate Jaeger telemetry with KrakenD API Gateway for distributed tracing and monitoring of your microservices architecture
weight: 110
notoc: false
menu:
  community_v2.10:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-otel
  namespace:
  - telemetry/opentelemetry
  scope:
  - service
  log_prefix:
  - "[SERVICE: OpenTelemetry]"
---
The KrakenD exporter to [Jaeger](https://www.jaegertracing.io/) allows you to submit spans to an OpenTelemetry Collector (HTTP or gRPC) automatically.

Jaeger is an open-source, end-to-end distributed tracing system that allows you to monitor and troubleshoot transactions in complex distributed systems. Use Jaeger when you want to see the complete flow of a user request through KrakenD and its connected services.

## Jaeger configuration
To add Jaeger, configure a new exporter to the [OpenTelemetry settings](/docs/v2.11/v2.10/telemetry/opentelemetry/). For instance:

```json
{
    "version": 3,
    "extra_config": {
        "telemetry/opentelemetry": {
            "service_name": "my_krakend_service",
            "metric_reporting_period": 1,
            "trace_sample_rate": 0.15,
            "layers": {
                "global": {
                    "report_headers": true
                },
                "proxy": {
                    "report_headers": true
                },
                "backend": {
                    "metrics": {
                        "disable_stage": true
                    },
                    "traces": {
                        "disable_stage": false,
                        "round_trip": true,
                        "read_payload": true,
                        "detailed_connection": true,
                        "report_headers": true
                    }
                }
            },
            "exporters": {
                "otlp": [
                    {
                        "name": "local_jaeger",
                        "host": "jaeger",
                        "port": 4317,
                        "use_http": false,
                        "disable_metrics": true
                    }
                ]
            }
        }
    }
}
```
The fields relative to the exporter are:

{{< schema version="v2.10" data="telemetry/opentelemetry.json" property="exporters" filter="otlp">}}

But as you can see there is a `layers` attribute in the example configuration that defines settings for all exporters (not only Jaeger). See the [layers options](/docs/v2.11/v2.10/telemetry/opentelemetry/#layers).

Also notice that port `4317` and `"use_http": false` are set, meaning that gRPC communication is used. Change to `4318` and the flag to `true` for HTTP communication.

## Jaeger demo environment
You can test this setup by running the **All in One** official Jaeger image and opening the necessary ports. For instance:
```yaml
version: "3"
services:
  krakend:
    image: {{< product image >}}:2.10
    volumes:
      - "./:/etc/krakend"
    ports:
      - "8080:8080"
  jaeger:
    image: jaegertracing/all-in-one:1.54
    environment:
      COLLECTOR_ZIPKIN_HOST_PORT: ":9411"
    ports:
      - "5778:5778" # serve configs
      - "16686:16686" # serve frontend UI
      - "4317:4317"   # otlp grpc: we remap this to be able to run other envs
      - "4318:4318"   # otlp http: we remap this to be able to run other envs
    deploy:
      resources:
        limits:
          memory: 4096M # Adjust according to your setup
```