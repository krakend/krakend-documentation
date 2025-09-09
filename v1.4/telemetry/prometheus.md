---
lastmod: 2021-06-13
old_version: true
date: 2019-09-15
notoc: true
linktitle: Prometheus
title: Exporting metrics to Prometheus
weight: 70
source: https://github.com/krakend/krakend-opencensus
menu:
  community_v1.4:
    parent: "080 Telemetry"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
[Prometheus](https://prometheus.io/) is an open-source systems monitoring and alerting toolkit.

The Opencensus exporter allows you to expose data to Prometheus. Enabling it only requires you to include in the root level of your configuration the Opencensus middleware with the `prometheus` exporter. Specify the `port` which Prometheus should hit, the `namespace` (optional), and Prometheus will start receiving the data.
{{< highlight json >}}
{
  "version": 2,
  "extra_config": {
    "github_com/devopsfaith/krakend-opencensus": {
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
{{< /highlight >}}

- `port` on which the Prometheus exporter should listen
- `namespace` sets the domain the metric belongs to. Optional field.

Optional fields (default to `false`):

- `tag_host` (*bool*): Whether to send the host as a metric or not.
- `tag_path` (*bool*): Whether to send the path as a metric or not. Client metrics are reported as: `/hello/:hello/:world` and backend metrics are reported as: `/{hello}/{world}`. Paths are case insensitive, all metrics are reported lowercased.
- `tag_method` (*bool*): Whether to send the HTTP method as a metric or not.
- `tag_statuscode` (*bool*): Whether to send the status code as a metric or not.

See also the [additional settings](/docs/v1.4/telemetry/opencensus/) of the Opencensus module that can be declared.
