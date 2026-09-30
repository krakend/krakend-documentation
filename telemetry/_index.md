---
lastmod: 2026-08-18
date: 2019-09-15
notoc: false
linktitle: Metrics and Traces overview
title: Telemetry and Monitoring
description: Learn about the telemetry and monitoring capabilities of KrakenD API Gateway, enabling real-time visibility and analysis of API performance
weight: 10
images:
- /images/documentation/available-exporters.png
dark_header_image: true
aliases: ["/docs/logging-metrics-tracing/tracing/", "/docs/telemetry/overview/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
---
KrakenD’s **no-lock-in philosophy** underscores flexibility and interoperability, guaranteeing that technology-specific dependencies don’t constrain you. Giving you choices for observability and networking tools is key to success in a distributed microservices architecture.

Our [OpenTelemetry](/docs/telemetry/opentelemetry/) integration provides:

- Ways to export data to detect root causes of problems.
- Monitoring and details of the different distributed transactions.
- Performance and latency optimization in the systems of your choice.
- The flexibility to use the monitoring system that you have chosen, and not one that you are locked into

**OpenTelemetry** is a unified, open-source framework for collecting and managing telemetry data over distributed systems, including traces and metrics. It offers vendor neutrality, simplifies instrumentation, and avoids lock-in with specific monitoring platforms. If you are fed up with a provider, you can simply push the metrics elsewhere by setting the new location in the [OTEL configuration](/docs/telemetry/opentelemetry/).

## OpenTelemetry integrations
As **OpenTelemetry is an open standard**, any provider that adopts it via the wire protocol will automatically be compatible with KrakenD. All major vendors natively support it, in different degrees of functionality.

You can choose between self-hosting your metrics/traces solution or using one of the many SaaS systems.

As providers and software makers make an ongoing effort to adopt OpenTelemetry, you can find an extensive list of systems.See the [vendors who natively support OpenTelemetry](https://opentelemetry.io/ecosystem/vendors/).

### Self-hosted systems using OTEL
When you want to have complete control of your metrics and traces, this is a list of software you can install in your infrastructure:

- **Prometheus**: An open-source system monitoring and alerting toolkit.
- **Jaeger**: An open-source, self-hosted solution for distributed tracing.
- **Elastic APM**: Part of the Elastic Stack, can be self-hosted for full control over data and infrastructure.
- **Grafana Tempo**: Integrates with Grafana can be self-hosted for tracing data.

All the options above allow you to have full ownership of the data.

### SaaS systems using OTEL
If you want to use a third-party SaaS and delegate the data to a vendor, here is a non-exhaustive list of a few APM systems that vary in their specific offerings, such as AI capabilities, ease of integration, visualization tools, and support for different programming languages and frameworks:

- New Relic
- Datadog
- Dynatrace
- Splunk APM (formerly SignalFx)
- AppDynamics (Cisco)
- Elastic APM
- Instana
- Google Cloud’s operations suite (formerly Stackdriver)
- AWS X-Ray
- Azure Monitor
- Jaeger
- Lightstep
- Honeycomb.io
- Sumo Logic
- SolarWinds AppOptics
- LogicMonitor
- Scout APM
- Rollbar
- Wavefront by VMware

The adoption of OpenTelemetry by these platforms indicates a strong industry shift towards standardized, open-source observability solutions. **You should test the ones you need on KrakenD** and ensure they deliver what you seek; from KrakenD, we are unfamiliar with every vendor out there.

{{< button-group >}}
{{< button url="/docs/telemetry/opentelemetry/" text="Configure OpenTelemetry" >}}<svg data-slot="icon" aria-hidden="true" fill="none" stroke-width="1.5" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
  <path d="M3.75 3v11.25A2.25 2.25 0 0 0 6 16.5h2.25M3.75 3h-1.5m1.5 0h16.5m0 0h1.5m-1.5 0v11.25A2.25 2.25 0 0 1 18 16.5h-2.25m-7.5 0h7.5m-7.5 0-1 3m8.5-3 1 3m0 0 .5 1.5m-.5-1.5h-9.5m0 0-.5 1.5m.75-9 3-3 2.148 2.148A12.061 12.061 0 0 1 16.5 7.605" stroke-linecap="round" stroke-linejoin="round"></path>
</svg>
{{< /button >}}
{{< /button-group >}}
