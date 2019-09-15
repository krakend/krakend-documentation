---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Google Stackdriver
title: Exporting metrics, logs and events to Google Stackdriver
weight: 120
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
menu:
  documentation:
    parent: logging-metrics-tracing
---

[Google Stackdriver](https://cloud.google.com/stackdriver/) aggregates metrics, logs, and events from infrastructure, giving developers and operators a rich set of observable signals that speed root-cause analysis and reduce mean time to resolution (MTTR).

The Opencensus exporter allows you export data to Google Stackdriver. Enabling it only requires you to add the `stackdriver` exporter in the [opencensus module](/docs/logging-metrics-tracing/opencensus/).

The following configuration snippet sends data to your X-Ray:

    "github_com/devopsfaith/krakend-opencensus": {
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

*   `project_id`: The identifier of your Google Cloud project.
*   `metrics_prefix`: A prefix that you can add to all your metrics for better organization.
*   `default_labels`: Enter here any label that will be assigned by default to the reported metric so you can filter later on Stack Driver. In the example we set a label `env` with the value `production`.

See also the [additional settings](/docs/logging-metrics-tracing/opencensus/) of the Opencensus module that can be declared.
