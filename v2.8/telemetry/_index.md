---
lastmod: 2024-03-08
old_version: true
date: 2019-09-15
notoc: false
linktitle: Metrics and Traces overview
title: Telemetry and Monitoring
description: Learn about the telemetry and monitoring capabilities of KrakenD API Gateway, enabling real-time visibility and analysis of API performance
weight: 10
images:
- /images/documentation/available-exporters.png
menu:
  community_v2.8:
    parent: "160 Monitoring, Logs, and Analytics"
---
KrakenD's **no lock-in philosophy** emphasizes flexibility and interoperability, ensuring technology-specific dependencies don't constrain you. Giving you choices for observability and networking tools is key to success in a distributed microservices architecture.

Our [OpenTelemetry](/docs/v2.8/telemetry/opentelemetry/) integration and its previous predecessors [OpenCensus](/docs/v2.8/telemetry/opencensus/), and the [Metrics API](/docs/v2.8/telemetry/extended-metrics/), are part of this effort. Our components provide:

- Ways to export data to detect root causes of problems.
- Monitoring and details of the different distributed transactions.
- Performance and latency optimization in the systems of your choice.
- The flexibility to use the monitoring system that you have chosen, and not one that you are locked-in

**OpenTelemetry** presents a unified, open-source framework for collecting and managing telemetry data across distributed systems, such as traces and metrics. It offers **vendor neutrality**,  **simplifies instrumentation**, and **avoids lock-in** with specific monitoring platforms. You can choose between **over twenty different providers**. [Add OTEL to your configuration](/docs/v2.8/telemetry/opentelemetry/).

**OpenCensus** is the previous component which has provided reliable service for over six years for traces and metrics, and now its development is frozen in favour of OpenTelemetry. You can still use [OpenCensus telemetry](/docs/v2.8/telemetry/opencensus/), although we recommend you to plan a transition to OpenTelemetry.

The **Metrics API** and its native exporter to InfluxDB are in a similar situation. It was our richest exporter of metrics data until the OpenTelemetry release, and while it still works, its development has also frozen. [See the Metrics API](/docs/v2.8/telemetry/extended-metrics/)

If starting with a new project, choose an OpenTelemetry integration for metrics and traces.

## OpenTelemetry integrations
As OpenTelemetry is **an open standard**, any provider adopting it using the wire protocol will automatically be compatible with KrakenD. More than 50 vendors natively support it.

So the question of *does KrakenD support provider X?* can be answered with another question: *Does your provider offer an OpenTelemetry integration?*.

As providers and software makers make an ongoing effort to adopt OpenTelemetry, you can find an extensive list of systems, SaaS or on-premise, that are on this path, and more are coming (see [vendors who natively support OpenTelemetry](https://opentelemetry.io/ecosystem/vendors/)).

If you work with KrakenD and a piece of software that is not in the list below, please add it!

### Self-hosted systems using OTEL
When you want to have complete control of your metrics and traces, this is a list of software you can install in your infrastructure:

- **Prometheus**: An open-source system monitoring and alerting toolkit.
- **Jaeger**: An open-source, self-hosted solution for distributed tracing.
- **Elastic APM**: Part of the Elastic Stack, can be self-hosted for full control over data and infrastructure.
- **Grafana Tempo**: Integrates with Grafana can be self-hosted for tracing data.

### SaaS systems using OTEL
If you want to use a third-party SaaS, here is a list of a few APM systems that vary in their specific offerings, such as AI capabilities, ease of integration, visualization tools, and support for different programming languages and frameworks:

- **New Relic**: Offers comprehensive monitoring with native support for OpenTelemetry.
- **Datadog**: Provides extensive analytics and monitoring, supporting OpenTelemetry protocols.
- **Dynatrace**: Known for AI-powered analytics and robust OpenTelemetry integration.
- **Splunk APM (formerly SignalFx)**: Offers real-time analytics and visualization compatible with OpenTelemetry.
- **AppDynamics (Cisco)**: Supports OpenTelemetry, providing performance analysis and proactive alerting.
- **Elastic APM**: Part of the Elastic Stack integrates well with OpenTelemetry.
- **Instana**: Offers automated APM for microservices with OpenTelemetry support.
- **Google Cloud’s operations suite (formerly Stackdriver)**: Provides an integrated suite for monitoring, logging, and diagnostics, compatible with OpenTelemetry.
- **AWS X-Ray**: Supports OpenTelemetry for applications running in AWS environments.
- **Azure Monitor**: Fully compatible with OpenTelemetry for monitoring applications on Azure.
- **Jaeger**: An open-source platform for distributed tracing that supports OpenTelemetry.
- **Lightstep**: Delivers detailed insights and is compatible with the OpenTelemetry protocol.
- **Honeycomb.io**: Emphasizes understanding production systems and supports OpenTelemetry.
- **Sumo Logic**: Offers cloud-native solutions with OpenTelemetry integration.
- **SolarWinds AppOptics**: Combines APM features with cloud monitoring, supporting OpenTelemetry.
- **LogicMonitor**: Known for its automated monitoring solutions that are compatible with OpenTelemetry.
- **Scout APM**: A developer-centric monitoring tool that supports OpenTelemetry.
- **Rollbar**: Focuses on real-time error tracking and debugging with OpenTelemetry support.
- **Wavefront by VMware**: A streaming analytics platform integrating with OpenTelemetry.

The adoption of OpenTelemetry by these platforms indicates a strong industry shift towards standardized, open-source observability solutions. **You should test the ones you need on KrakenD** and ensure they deliver what you seek; from KrakenD, we are unfamiliar with every vendor out there.

{{< button-group >}}
{{< button url="/docs/telemetry/opentelemetry/" text="Configure OpenTelemetry" >}}<svg data-slot="icon" aria-hidden="true" fill="none" stroke-width="1.5" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <path d="M3.75 3v11.25A2.25 2.25 0 0 0 6 16.5h2.25M3.75 3h-1.5m1.5 0h16.5m0 0h1.5m-1.5 0v11.25A2.25 2.25 0 0 1 18 16.5h-2.25m-7.5 0h7.5m-7.5 0-1 3m8.5-3 1 3m0 0 .5 1.5m-.5-1.5h-9.5m0 0-.5 1.5m.75-9 3-3 2.148 2.148A12.061 12.061 0 0 1 16.5 7.605" stroke-linecap="round" stroke-linejoin="round"></path>
</svg>
{{< /button >}}
{{< /button-group >}}
