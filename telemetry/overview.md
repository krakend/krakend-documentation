---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Telemetry integrations
title: Telemetry
weight: 30
images:
- /images/documentation/exporters.png
aliases: ["/docs/logging-metrics-tracing/tracing/"]
menu:
  community_current:
    parent: "080 Telemetry"
---
Observability and networking are key to succeed in a scenario of distributed microservices architecture, and new monitoring tools are needed. These tools must provide at least options to detect problems' root causes, monitoring and details of the different distributed transactions, and performance and latency optimization.

Through the [OpenCensus exporters](/docs/telemetry/opencensus/) you can send logs, metrics, and traces to several open source and payment tools, so you can follow the activity of the gateway and the derived requests to its connected backends.

Enabling tracing is key to have a good detail of what is going on inside the Gateway and between the user, the Gateway and your services.

The following telemetry systems are supported:

- [Jaeger](/docs/telemetry/jaeger/)
- [Zipkin](/docs/telemetry/zipkin/)
- [AWS X-Ray](/docs/telemetry/xray/)
- [InfluxDB](/docs/telemetry/influxdb/)
- [Prometheus](/docs/telemetry/prometheus/)
- [Google Stackdriver](/docs/telemetry/stackdriver/)
- [Datadog](/docs/telemetry/datadog/)
- [OpenCensus Agent](/docs/telemetry/opencensus/) (e.g: **Azure Monitor**)
- [Grafana](/docs/extended-metrics/grafana/)
- Newrelic (only custom builds)
- Instana (KrakenD Enterprise only)
