---
lastmod: 2022-09-22
date: 2019-09-15
notoc: true
linktitle: Google Cloud
title: Exporting metrics and traces to Google Cloud's operations suite
weight: 120
notoc: true
aliases: ["/docs/logging-metrics-tracing/stackdriver/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.7
  source: https://github.com/krakendio/krakend-opencensus
  namespace:
  - telemetry/opencensus
  log_prefix:
  - "[SERVICE: Opencensus]"
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---

[Google Cloud's operation suite](https://cloud.google.com/products/operations) (formerly [Stackdriver](https://cloud.google.com/stackdriver/)) aggregates metrics, logs, and events from infrastructure, giving developers and operators a rich set of observable signals that speed root-cause analysis and reduce mean time to resolution (MTTR).

The Opencensus exporter allows you to export **metrics and traces** to Google Cloud. Enabling it only requires you to add the `stackdriver` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends the data:

{{< highlight json >}}
{
  "extra_config": {
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 60,
      "enabled_layers": {
        "backend": true,
        "router": true,
        "pipe": true
      },
      "exporters": {
        "stackdriver": {
          "project_id": "my-krakend-project",
          "metrics_prefix": "krakend",
          "default_labels": {
          "env": "production"
          }
        }
      }
    }
  }
}
{{< /highlight >}}

- `sample_rate` is the percentage of sampled requests. A value of `100` means that all requests are exported (100%). If you are processing a considerable amount of traffic, you might want to sample only a part of what's happening.
- `reporting_period` is the number of **seconds** passing between reports. It must be **`60` or greater**, otherwise, Google will reject the connection.
- `exporters` is a key-value with all the exporters you want to use. See each exporter configuration for the underlying keys.
  - `project_id`: The identifier of your Google Cloud project. The `project_id` **is not the project name**. You can omit this value from the configuration if you have an application credential file for Google.
  - `metrics_prefix`: A prefix that you can add to all your metrics for better organization.
  - `default_labels`: Enter any label assigned by default to the reported metric so you can filter later on Stack Driver. In the example, we set the label `env` with the value `production`.
- `enabled_layers` let you specify what data you want to export. All layers are enabled by default:
  - Use `backend` to report the activity between KrakenD and your services
  - Use `router` to report the activity between end-users and KrakenD
  - Use `pipe` to report the activity at the beginning of the proxy layer. It gives a more detailed view of the internals of the pipe between end-users and KrakenD.

## Authentication to Google Cloud
The exporter searches for the **Application Default Credentials**. It looks for credentials in the following places, preferring the first location found:

1. A JSON file whose path is specified by the `GOOGLE_APPLICATION_CREDENTIALS` environment variable.
2. A JSON file in a location known to the `gcloud` command-line tool (e.g.: `$HOME/.config/gcloud/application_default_credentials.json`).
3. On Google Compute Engine and Google App Engine flexible environment, it fetches credentials from the metadata server.