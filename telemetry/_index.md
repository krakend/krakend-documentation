---
lastmod: 2022-10-24
date: 2019-09-15
notoc: true
linktitle: Metrics and Traces overview
title: Telemetry and Monitoring in KrakenD API Gateway
description: Learn about the telemetry and monitoring capabilities of KrakenD API Gateway, enabling real-time visibility and analysis of API performance
weight: 10
images:
- /images/documentation/available-exporters.png
aliases: ["/docs/logging-metrics-tracing/tracing/", "/docs/telemetry/overview/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
---
Observability and networking are key to succeed in a scenario of distributed microservices architecture, and new monitoring tools are needed. These tools must provide at least options to detect problems' root causes, monitoring and details of the different distributed transactions, and performance and latency optimization.

Through the [OpenCensus exporters](/docs/telemetry/opencensus/) you can send metrics, and traces to several open source and proprietary tools, so you can follow the activity of the gateway and the derived requests to its connected backends. Although you can also do logging with OpenCensus, we recommend using the [standard logging](/docs/logging/)component.

Enabling metrics and traces is key to have a good detail of what is going on inside the Gateway and between the user, the Gateway and your services.

The following telemetry third-party systems are supported:

- [Jaeger](/docs/telemetry/jaeger/)
- [Zipkin](/docs/telemetry/zipkin/)
- [AWS X-Ray](/docs/telemetry/xray/)
- [InfluxDB](/docs/telemetry/influxdb/)
- [Prometheus](/docs/telemetry/prometheus/)
- [Google Stackdriver](/docs/telemetry/stackdriver/)
- [Datadog](/docs/telemetry/datadog/)
- [Azure Monitor](/docs/telemetry/azure/)
- [Grafana](/docs/telemetry/grafana/)
- [Newrelic](/docs/enterprise/telemetry/newrelic/) {{< badge color="denim" >}}Enterprise{{< /badge >}}
