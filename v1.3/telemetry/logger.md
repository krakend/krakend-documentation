---
lastmod: 2019-09-15
old_version: true
date: 2019-09-15
canonical: /docs/v2.5/telemetry/logger/
notoc: true
linktitle: Logger
title: Exporting to the logger
weight: 1000
source: https://github.com/krakend/krakend-opencensus
notoc: true
menu:
  community_v1.3:
    parent: "080 Telemetry"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
Opencensus can export data to the system logger as another exporter.

Enabling it only requires you to add the `logger` exporter in the [opencensus module](/docs/v1.3/telemetry/opencensus/).

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


See also the [additional settings](/docs/v1.3/telemetry/opencensus/) of the Opencensus module that can be declared.
