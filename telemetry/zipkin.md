---
lastmod: 2024-02-27
date: 2019-09-15
notoc: true
linktitle: Zipkin
title: Zipkin Telemetry Integration
description: Integrate Zipkin telemetry to monitor and trace KrakenD API Gateway requests efficiently for enhanced performance
weight: 100
aliases: ["/docs/logging-metrics-tracing/zipkin/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-otel
  namespace:
  - telemetry/opentelemetry
  log_prefix:
  - "[SERVICE: OpenTelemetry]"
  scope:
  - service
---
[Zipkin](https://zipkin.io/) is a distributed tracing system. It helps gather timing data needed to troubleshoot latency problems in service architectures.

The OpenTelemetry exporter allows you export data to Zipkin via a [zipkin-otel collector](https://github.com/openzipkin-contrib/zipkin-otel). Once you have the zipkin-otel running, add in the KrakenD configuration an`otlp` exporter in the [OpenTelemetry module](/docs/telemetry/opentelemetry/) with the port and protocol you have chosen.

See the [OpenTelemetry configuration](/docs/telemetry/opentelemetry/).