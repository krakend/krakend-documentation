---
lastmod: 2022-10-24
old_version: true
date: 2019-09-15
notoc: true
linktitle: Telemetry integrations
title: Telemetry
weight: 1
images:
- /images/documentation/available-exporters-old.png
menu:
  community_v2.1:
    parent: "080 Telemetry and Analytics"
---
Observability and networking are key to succeed in a scenario of distributed microservices architecture, and new monitoring tools are needed. These tools must provide at least options to detect problems' root causes, monitoring and details of the different distributed transactions, and performance and latency optimization.

Through the [OpenCensus exporters](/docs/v2.1/telemetry/opencensus/) you can send logs, metrics, and traces to several open source and payment tools, so you can follow the activity of the gateway and the derived requests to its connected backends.

Enabling tracing is key to have a good detail of what is going on inside the Gateway and between the user, the Gateway and your services.

The following telemetry third-party systems are supported:

- [Jaeger](/docs/v2.1/telemetry/jaeger/)
- [Zipkin](/docs/v2.1/telemetry/zipkin/)
- [AWS X-Ray](/docs/v2.1/telemetry/xray/)
- [InfluxDB](/docs/v2.1/telemetry/influxdb/)
- [Prometheus](/docs/v2.1/telemetry/prometheus/)
- [Google Stackdriver](/docs/v2.1/telemetry/stackdriver/)
- [Datadog](/docs/v2.1/telemetry/datadog/)
- [Azure Monitor](/docs/v2.1/telemetry/opencensus/) (OpenCensus Agent)
- [Grafana](/docs/v2.1/telemetry/grafana/)

Plus the Enterprise-only exporters:

- [Newrelic](/docs/enterprise/v2.1/telemetry/newrelic/)
- [Instana](/docs/enterprise/v2.1/telemetry/instana/)
- [Google Analytics](/docs/enterprise/v2.1/telemetry/google-analytics/)
