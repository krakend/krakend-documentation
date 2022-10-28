---
lastmod: 2022-10-24
date: 2019-09-15
notoc: true
linktitle: Telemetry integrations
title: Telemetry
weight: 1
images:
- /images/documentation/exporters.png
aliases: ["/docs/logging-metrics-tracing/tracing/", "/docs/telemetry/overview/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
---
Observability and networking are key to succeed in a scenario of distributed microservices architecture, and new monitoring tools are needed. These tools must provide at least options to detect problems' root causes, monitoring and details of the different distributed transactions, and performance and latency optimization.

Through the [OpenCensus exporters](/docs/telemetry/opencensus/) you can send logs, metrics, and traces to several open source and payment tools, so you can follow the activity of the gateway and the derived requests to its connected backends.

Enabling tracing is key to have a good detail of what is going on inside the Gateway and between the user, the Gateway and your services.

The following telemetry third-party systems are supported:

- [Jaeger](/docs/telemetry/jaeger/)
- [Zipkin](/docs/telemetry/zipkin/)
- [AWS X-Ray](/docs/telemetry/xray/)
- [InfluxDB](/docs/telemetry/influxdb/)
- [Prometheus](/docs/telemetry/prometheus/)
- [Google Stackdriver](/docs/telemetry/stackdriver/)
- [Datadog](/docs/telemetry/datadog/)
- [Azure Monitor](/docs/telemetry/opencensus/) (OpenCensus Agent)
- [Grafana](/docs/telemetry/grafana/)

Plus the Enterprise-only exporters:

- [Newrelic](/docs/enterprise/telemetry/newrelic/)
- [Instana](/docs/enterprise/telemetry/instana/)
- [Google Analytics](/docs/enterprise/telemetry/google-analytics/)