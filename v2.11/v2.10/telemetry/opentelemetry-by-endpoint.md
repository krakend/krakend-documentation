---
lastmod: 2024-10-18
old_version: true
old_version: true
date: 2024-10-18
linktitle: OpenTelemetry by Endpoint
title: Granular OpenTelemetry by endpoint
description: Set different metrics and traces settings per endpoint or per backend individually, overriding existing OpenTelemetry settings that are defined at the service level
weight: 22
notoc: true
images:
- /images/documentation/diagrams/opentelemetry-otlp.mmd.svg
skip_header_image: true
menu:
  community_v2.10:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: v2.8
  namespace:
  - telemetry/opentelemetry
  scope:
  - endpoint
  - backend
  log_prefix:
  - "[SERVICE: OpenTelemetry]"
---

The OpenTelemetry configuration is declared [at the service level](/docs/v2.11/v2.10/telemetry/opentelemetry/), but you can override metrics and traces per endpoint and per backend as follows.

## Endpoint override of metrics and traces
The following example overrides properties that could be declared at the service level.

```json
{
    "endpoints": [
        {
            "endpoint": "/example",
            "backend": [
                {
                    "host": [
                        "example.com"
                    ],
                    "url_pattern": "/example"
                }
            ],
            "extra_config": {
                "telemetry/opentelemetry": {
                    "proxy": {
                        "disable_metrics": false,
                        "disable_traces": false,
                        "report_headers": true,
                        "traces_static_attributes": [
                          {
                            "key": "owner",
                            "value": "team-charlie"
                          }
                        ],
                        "metrics_static_attributes": [
                          {
                            "key": "owner",
                            "value": "team-charlie"
                          }
                        ]
                    }
                }
            }
        }
    ]
}
```

The full list of options is:

{{< schema version="v2.10" data="telemetry/opentelemetry-endpoint.json" filter="proxy" title="OpenTelemetry settings per endpoint">}}

In the Enterprise Edition you have more override options, like [override entirely the exporter you want to use](/docs/enterprise/telemetry/opentelemetry-by-endpoint/).

## Backend override of metrics and traces
For instance, you have a specific backend that is adding noise to your dashboards and you'd like to disable all layers:

```json
{
    "backend": [
        {
            "extra_config": {
                "telemetry/opentelemetry": {
                    "proxy": {
                        "disable_metrics": true,
                        "disable_traces": true
                    },
                    "backend": {
                        "metrics": {
                            "disable_stage": true
                        },
                        "traces": {
                            "disable_stage": true
                        }
                    }
                }
            },
            "url_pattern": "/noise",
            "host": [
                "example.com"
            ]
        }
    ]
}
```
These are the options:

{{< schema version="v2.10" data="telemetry/opentelemetry-backend.json" title="OpenTelemetry settings per backend" >}}
