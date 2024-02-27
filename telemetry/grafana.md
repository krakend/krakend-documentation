---
lastmod: 2022-09-28
date: 2020-11-17
linktitle: Grafana Dashboard
title: Telemetry and Monitoring with Grafana
description: Discover how to set up telemetry and monitoring for KrakenD API Gateway using Grafana, gaining insights into performance and usage metrics
weight: 30
aliases: ["/docs/extended-metrics/grafana/"]
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
skip_header_image: false
images:
- /images/documentation/grafana-screenshot.png
meta:
  since: 0.5
  namespace:
  - telemetry/influx
  - telemetry/metrics
  scope:
  - service
---

The preconfigured Grafana dashboard for KrakenD offers valuable information to understand the performance of your services and detect anomalies in the service.

The dashboard is extensive and offers you metrics like:

- Requests from users to KrakenD
- Requests from KrakenD to your backends
- Response times
- Memory usage and details
- Endpoints and status codes
- Heatmaps
- Open connections
- Throughput
- Distributions, timers, garbage collection and a long etcetera

{{< youtube Ik18Zlwyap8 >}}

## Importing a Grafana dashboard
These are the different Grafana dashboards you can use depending on your data sources:

| Datasource | Description | Cloud ID | Source |
|----------|-----------|----------|----------|
| Prometheus<br>(**recommended**) | Latest dashboard to display metrics when you add an [OpenTelemetry Prometheus exporter](/docs/telemetry/prometheus/) | `xxx` | [for-prometheus.json](https://github.com/krakend/telemetry-dashboards/blob/main/grafana/krakend/xxx.json)|
| InfluxDB v2.x<br>(*legacy*) | Legacy dashboard for InfluxDB v2 and KrakenD under v2.6. Uses Flux queries | `17074` | [for-influxdb-v2.json](https://github.com/krakend/telemetry-dashboards/blob/main/grafana/krakend/for-influxdb-v2.json)|
| InfluxDB v1.x<br>(*legacy*)| Legacy dashboard for InfluxDB v1 and KrakenD under v2.6. Uses InfluxQL queries | `15029` | [for-influxdb-v1.json](https://github.com/krakend/telemetry-dashboards/blob/main/grafana/krakend/for-influxdb-v1.json)|

To import them, there are several options, being the most common:

- From your Grafana UI, click the `+` icon in the side menu, and then click *Import*. Choose import via Grafana.com and use the IDs above.
- From the same UI, import the JSON source files instead
- Copy or mount in your Grafana container the dashboards when starting ([Volume content here](https://github.com/krakend/telemetry-dashboards)):
```yml
volumes:
  - "./grafana/datasources/all.yml:/etc/grafana/provisioning/datasources/all.yml"
  - "./grafana/dashboards/all.yml:/etc/grafana/provisioning/dashboards/all.yml"
  - "./grafana/krakend:/var/lib/grafana/dashboards/krakend"
```
 You can see an example integrated on [KrakenD Playground](https://github.com/krakend/playground-community)'s Docker compose file.

## Getting the metrics on Grafana
Grafana does not require any specific configuration on KrakenD, but it feeds from a data source, so you will need to push data to one of the following:

1. [Prometheus exporter](/docs/telemetry/prometheus/) (**recommended**)
2. [Legacy integration for InfluxDb](/docs/telemetry/influxdb/#legacy-integration), for older versions of KrakenD
