---
lastmod: 2024-02-27
date: 2019-09-15
linktitle: InfluxDB
title: InfluxDB Telemetry Integration
description: Integrate InfluxDB telemetry with KrakenD API Gateway for efficient data collection, storage, and visualization of API performance metrics
weight: 50
#notoc: true
aliases: ["/docs/extended-metrics/influxdb/","/docs/logging-metrics-tracing/influxdb/","/docs/telemetry/influxdb-native/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
images:
  - /images/documentation/diagrams/opentelemetry-otlp-influx.mmd.svg
skip_header_image: true
notoc: true
meta:
  since: 0.5 #telemetry/influxdb was released on 0.5
  source: https://github.com/krakend/krakend-otel
  namespace:
  - telemetry/opentelemetry
  scope:
  - service
  log_prefix:
  - "[SERVICE: OpenTelemetry]"
---
[InfluxDB](https://www.influxdata.com/) is a time series database designed to handle high write and query loads and allows you to store and visualize metrics data. Influx is offered as an open-source solution you can host but also as a cloud service.

Whatever flavor you use, KrakenD instruments your API automatically code and generates telemetry data that is pushed using the **OpenTelemetry Protocol** (OTLP) via the [OpenTelemetry integration](/docs/telemetry/opentelemetry/). The data can travel through gRPC or HTTP and uses the standard OTLP format an [OTEL Collector](https://opentelemetry.io/docs/collector/) expects.

The OpenTelemetry integration populates metrics data to an Influx database when you add the `telemetry/opentelemetry` namespace to the configuration with an `otlp` exporter. There is no specific setting for InfluxDB because OTEL is a standard shared with many other technologies.

{{< note title="Older influx component" type="info" >}}
Prior to KrakenD v2.6, Influx was configured using the native exporter `telemetry/influx`. This option is now deprecated, but the documentation is still available for the older versions.
{{< /note >}}


The flow for the Influx population using OTLP is as follows:

![KrakenD OTLP communication with a Collector that sends to Influx](/images/documentation/diagrams/opentelemetry-otlp-influx.mmd.svg)

## InfluxDB configuration
An example configuration is as follows:


{{< schema data="telemetry/opentelemetry.json" property="exporters" filter="otlp">}}
