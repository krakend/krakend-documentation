---
lastmod: 2019-09-15
old_version: true
date: 2019-09-15
notoc: true
linktitle: Logger
title: Exporting to the logger
weight: 1000
source: https://github.com/krakendio/krakend-opencensus
notoc: true
menu:
  community_v1.4:
    parent: "080 Telemetry"
meta:
  since: 0.5
  source: https://github.com/krakendio/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
Opencensus can export data to the system logger as another exporter.

Enabling it only requires you to add the `logger` exporter in the [opencensus module](/docs/v1.4/telemetry/opencensus/).

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


See also the [additional settings](/docs/v1.4/telemetry/opencensus/) of the Opencensus module that can be declared.
