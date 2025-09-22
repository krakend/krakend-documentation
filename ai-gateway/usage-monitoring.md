---
lastmod: 2025-09-17
date: 2025-09-17
linktitle: AI Monitoring
title: AI usage monitoring
description: Use OpenTelemetry for monitoring API gateway traffic, including AI calls, with your existing providers.
menu:
  community_current:
    parent: "100 AI Gateway"
weight: 1010
---

Monitoring AI backends in KrakenD does not require special treatment. AI services are integrated and handled just like any other backend in your API Gateway. Your existing monitoring tools and practices, such as [OpenTelemetry](/docs/telemetry/opentelemetry/), remain fully applicable.

AI backends, regardless of their complexity, respond to API calls just like any other backend service. KrakenD treats AI integrations as regular backends. This simplifies observability and operational consistency by applying proven monitoring frameworks without specialized or separate tooling.

## OpenTelemetry integration for AI backends
Use OpenTelemetry to monitor your API gateway traffic, including AI backend calls. The following example enables Jaeger traces and Prometheus metrics:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/opentelemetry": {
      "service_name": "krakend_prometheus_service",
      "metric_reporting_period": 1,
      "trace_sample_rate": 1,
      "exporters": {
        "prometheus": [
          {
            "name": "local_prometheus",
            "port": 9090,
            "process_metrics": true,
            "go_metrics": true
          }
        ],
        "otlp": [
          {
            "disable_metrics": false,
            "disable_traces": false,
            "host": "localhost",
            "name": "debug_jaeger",
            "port": 64317,
            "use_http": false
          }
        ]
      },
      "layers": {
        "global": {
          "disable_metrics": false,
          "disable_propagation": false,
          "disable_traces": false,
          "report_headers": true
        },
        "proxy": {
          "disable_metrics": false,
          "disable_traces": false,
          "report_headers": true
        },
        "backend": {
          "metrics": {
            "detailed_connection": true,
            "disable_stage": false,
            "read_payload": false,
            "round_trip": false
          },
          "traces": {
            "detailed_connection": false,
            "disable_stage": false,
            "read_payload": false,
            "report_headers": false,
            "round_trip": false
          }
        }
      }
    }
  }
}
```
