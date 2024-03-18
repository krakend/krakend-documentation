---
lastmod: 2019-09-15
old_version: true
date: 2019-09-15
canonical: /docs/v2.5/telemetry/logger/
notoc: true
linktitle: Logger
title: Exporting to the logger
weight: 1000
notoc: true
menu:
  community_v2.0:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
Opencensus can export data to the system logger as another exporter.

Enabling it only requires you to add the `logger` exporter in the [opencensus module](/docs/v2.0/telemetry/opencensus/).

The following configuration snippet enables the logger:
{{< highlight json >}}
{
  "extra_config":{
    "telemetry/opencensus": {
        "exporters": {
          "logger": {
              "stats": true,
              "spans": true
          }
        }
    }
}
{{< /highlight >}}

- `stats`: Whether to log the statistics or not
- `spans`: Whether to log the spans or not


See also the [additional settings](/docs/v2.0/telemetry/opencensus/) of the Opencensus module that can be declared.
