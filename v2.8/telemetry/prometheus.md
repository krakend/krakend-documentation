---
lastmod: 2024-03-08
old_version: true
date: 2019-09-15
notoc: true
linktitle: Prometheus
title: Prometheus' metrics endpoint
description: Learn how to integrate Prometheus telemetry with KrakenD API Gateway for efficient monitoring and performance analysis of your APIs
weight: 30
menu:
  community_v2.8:
    parent: "160 Monitoring, Logs, and Analytics"
images:
  - /images/documentation/screenshots/grafana-prometheus-otel.png
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
[Prometheus](https://prometheus.io/) is an open-source system monitoring and alerting toolkit that you can use to scrap a `/metrics` endpoint on KrakenD in the selected port. For instance, you could have an endpoint like `http://localhost:9091/metrics`.

When using Prometheus with OpenTelemetry, you can use a [ready-to-use Grafana dashboard](/docs/v2.8/telemetry/grafana/) to visualize metrics, as shown in the image above.

The mechanics are simple: you add the `telemetry/opentelemetry` integration with a `prometheus` exporter, and then you add a Prometheus job to scrap from your KrakenD instances the metrics.

![Prometheus scrapping from KrakenD image](/images/documentation/diagrams/opentelemetry-prometheus.mmd.svg)

## Prometheus Configuration
To enable scrapable Prometheus metrics on Krakend, add the [OpenTelemetry integration](/docs/v2.8/telemetry/opentelemetry/) with a `prometheus` exporter. The following configuration is an example of how to do it:

```json
{
    "version": 3,
    "extra_config": {
        "telemetry/opentelemetry": {
            "service_name": "krakend_prometheus_service",
            "metric_reporting_period": 1,
            "exporters": {
                "prometheus": [
                    {
                        "name": "local_prometheus",
                        "port": 9090,
                        "process_metrics": true,
                        "go_metrics": true
                    }
                ]
            }
        }
    }
}
```
The full list of the Prometheus exporter settings are as follows:

{{< schema version="v2.8" data="telemetry/opentelemetry.json" property="exporters" filter="prometheus" >}}

In addition, you can do a **granular configuration** of the metrics you want to expose using the `layers` attribute and other [OpenTelemetry options](/docs/v2.8/telemetry/opentelemetry/#layers).

### Demonstration setup
The following configuration allows you to test a complete metrics experience, from generation and collection to visualization. The first code snippet is a `docker-compose.yaml` that declares three different services:


- The `krakend` service exposing port 8080
- The `prometheus` service that will scrap the metrics from KrakenD
- A `grafana` dashboard to display them (it uses our [Grafana dashboard](/docs/v2.8/telemetry/grafana/))

Notice that the three services declare volumes to pick the configuration.

```yaml
version: "3"
services:
  krakend:
    image: "{{< product image >}}:2.8"
    ports:
      - "8080:8080"
    volumes:
      - "./krakend:/etc/krakend/"
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - "./prometheus.yml:/etc/prometheus/prometheus.yml"
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_USER: krakend
      GF_SECURITY_ADMIN_PASSWORD: krakend
      GF_AUT_ANONYMOUS_ENABLED: "true"
    volumes:
      - "./conf/provisioning/datasources:/etc/grafana/provisioning/datasources"
      - "./conf/provisioning/dashboards:/etc/grafana/provisioning/dashboards"
      - "./conf/data/dashboards:/var/lib/grafana/dashboards"
```

The following YAML configuration is a simple example of pulling data from the `/metrics` endpoint in KrakenD integration from three different instances:

{{< note title="Make sure ports are accessible" type="warning" >}}
To let the scrapper access the metrics endpoint, make sure that the path and the port are the ones you configured, that the listen address allows you to access the data, and that if you use containers, the port is exposed in KrakenD. Also, remember that you cannot use `localhost` as a target because the Prometheus container does not run inside the KrakenD container; use the service name instead.
{{< /note >}}


```yaml
global:
  scrape_interval:     15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:
  - job_name: krakend_otel
    scrape_interval: 5s
    metrics_path: '/metrics'
    static_configs:
      - targets:
        - 'krakend1:9091'
        - 'krakend2:9091'
        - 'krakend3:9091'
        labels:
          app: kotel_example
```
## Visualizing metrics in a dashboard
When the Prometheus configuration is added into KrakenD, and your Prometheus is scrapping it, you can visualize the data using our [Grafana dashboard](/docs/v2.8/telemetry/grafana/) or make your own.

{{< note title="Which layers do you need?" type="tip" >}}
Our Grafana dashboard contains a lot of options, and **not all are enabled by default**. Because generating low-detail metrics is an expensive operation, some options in the `layers` are disabled by default. Enable the options that matter to you, knowing that the more detail you add, the more resources the gateway will need to run.
{{< /note >}}

![Screenshot of a grafana dashboard with KrakenD metrics](/images/documentation/screenshots/grafana-prometheus-otel.png)

## Migrating from an old OpenCensus configuration (legacy)
Prior to KrakenD v2.6, you had to configure the Prometheus endpoint using the opencensus component. The OpenTelemetry integration is much more powerful and delivers more data while simultaneously giving you more configuration options.

If you had an OpenCensus configuration with a `prometheus` exporter like the following:
```json
{
  "version": 3,
  "extra_config": {
    "telemetry/opencensus": {
        "sample_rate": 100,
        "reporting_period": 0,
        "exporters": {
          "prometheus": {
              "port": 9091,
              "namespace": "krakend",
              "tag_host": false,
              "tag_path": true,
              "tag_method": true,
              "tag_statuscode": false
          }
      }
    }
  }
}
```

Then you should make the following changes to upgrade:

- `telemetry/opencensus` -> Rename to `telemetry/opentelemetry`
- `sample_rate` -> Delete this field
- `reporting_period` -> Rename to `metric_reporting_period`
- `prometheus: {...}` -> Add an array surrounding the object, so it becomes `prometheus: [{...}]`
- `namespace` -> Rename to `name`
- `tag_host`, `tag_path`,`tag_method`,`tag_statuscode` -> Delete them

From here, add any of the additional properties you can add.
