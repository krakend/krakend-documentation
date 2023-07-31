---
lastmod: 2022-10-24
old_version: true
date: 2019-09-15
notoc: true
linktitle: Google Cloud
title: Exporting metrics and traces to Google Cloud's operations suite
weight: 120
notoc: true
menu:
  community_v2.2:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.7
  source: https://github.com/krakend/krakend-opencensus
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

The Opencensus exporter allows you to export **metrics and traces** to Google Cloud. Enabling it only requires you to add the `stackdriver` exporter in the [opencensus module](/docs/v2.2/telemetry/opencensus/).

The following configuration snippet sends the data:

```json
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
          "metric_prefix": "krakend",
          "default_labels": {
            "env": "production"
          }
        }
      }
    }
  }
}
```

As with all [OpenCensus exporters](/docs/v2.2/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.2" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `stackdriver` entry with the following properties:

{{< schema version="v2.2" data="telemetry/opencensus.json" property="exporters" filter="stackdriver" >}}

See also the [additional settings](/docs/v2.2/telemetry/opencensus/) of the Opencensus module that can be declared.

{{< note title="Google does not accept low reporting periods" type="warning" >}}
The number of **seconds** passing between reports in `reporting_period` must be **`60` or greater**, otherwise, Google will reject the connection.
{{< /note >}}

## Authentication to Google Cloud
The exporter searches for the **Application Default Credentials**. It looks for credentials in the following places, preferring the first location found:

1. A JSON file whose path is specified by the `GOOGLE_APPLICATION_CREDENTIALS` environment variable.
2. A JSON file in a location known to the `gcloud` command-line tool (e.g.: `$HOME/.config/gcloud/application_default_credentials.json`).
3. On Google Compute Engine and Google App Engine flexible environment, it fetches credentials from the metadata server.