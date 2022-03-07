---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Graylog - GELF
title: Graylog and the GELF format
weight: 40
aliases: ["/docs/logging-metrics-tracing/graylog-gelf/"]
notoc: true
menu:
  community_current:
    parent: "090 Logging"
meta:
  since: 0.7
  source: https://github.com/devopsfaith/krakend-gelf
  namespace:
  - telemetry/gelf
  scope:
  - service
log_prefix:
  - "[SERVICE: Logging][GELF]"
---
KrakenD supports sending structured events in GELF format to your Graylog Cluster thanks to the [krakend-gelf](https://github.com/devopsfaith/krakend-gelf) integration.

The setup of GELF is straightforward and requires only to set two parameters:

- `address`: The address (including the port) of your Graylog cluster (or any other service that receives GELF inputs).
- `enable_tcp`: Set to `false` (recommended) to use UDP. When using TCP performance might be affected.

## Enabling GELF
Add the `krakend-gelf` integration in the root level of your `krakend.json`, inside the `extra_config` section. **The `gologging` needs to be enabled too**.

For instance:
{{< highlight json >}}
{
    "extra_config": {
      "telemetry/gelf": {
        "address": "myGraylogInstance:12201",
        "enable_tcp": false
      },
      "telemetry/logging": {
          "level": "INFO",
          "prefix": "[KRAKEND]",
          "syslog": false,
          "stdout": true
      }
    }
}
{{< /highlight >}}