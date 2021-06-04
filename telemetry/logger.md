---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Logger
title: Exporting to the logger
weight: 1000
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
aliases:
- /docs/logging-metrics-tracing/logger/
menu:
  community_current:
    parent: "080 Telemetry"
meta:
  since: 0.5
  source: https://github.com/devopsfaith/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
Opencensus can export data to the system logger as another exporter.

Enabling it only requires you to add the `logger` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet enables the logger:

    "github_com/devopsfaith/krakend-opencensus": {
        "exporters": {
          "logger": {
              "stats": true,
              "spans": true
          }
        }
    }

- `stats`: Whether to log the statistics or not
- `spans`: Whether to log the spans or not


See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.
