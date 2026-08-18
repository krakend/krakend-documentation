---
lastmod: 2026-08-18
date: 2019-09-15
linktitle: AWS X-Ray
title: AWS X-Ray Telemetry Integration
description: Push KrakenD metrics to AWS X-Ray Telemetry with a small snippet of configuration to monitor and analyze KrakenD API Gateway performance effectively
weight: 120
notoc: true
aliases: ["/docs/logging-metrics-tracing/xray/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  source: https://github.com/krakend/krakend-otel
  namespace:
  - telemetry/opentelemetry
  log_prefix:
  - "[SERVICE: OpenTelemetry]"
  scope:
  - service
---
[AWS X-Ray](https://aws.amazon.com/xray/) is a service offered by Amazon that provides an end-to-end view of requests as they travel through your application, and shows a map of your application’s underlying components.

The OpenTelemetry integration allows you export data to AWS X-Ray. For that, the [AWS Distro for OpenTelemetry](https://aws-otel.github.io/) Collector (ADOT Collector) is an AWS supported version of the upstream OpenTelemetry Collector and is distributed by Amazon. It enables users to send telemetry data to AWS CloudWatch Metrics, Traces, and Logs backends as well as the other supported backends.

Once you have configured it, add it to the [OpenTelemetry configuration block](/docs/telemetry/opentelemetry/)