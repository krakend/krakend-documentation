---
lastmod: 2022-10-25
old_version: true
date: 2019-09-15
notoc: true
linktitle: Graylog - GELF
title: Graylog and the GELF format
weight: 40
notoc: true
menu:
  community_v2.3:
    parent: "090 Logging"
meta:
  since: 0.7
  source: https://github.com/krakend/krakend-gelf
  namespace:
  - telemetry/gelf
  scope:
  - service
log_prefix:
  - "[SERVICE: Logging][GELF]"
---
KrakenD supports sending structured events in GELF format to your Graylog Cluster thanks to the [krakend-gelf](https://github.com/krakend/krakend-gelf) integration.

The setup of GELF is straightforward and requires to add **two components** in the configuration:

- `telemetry/logging` to capture the logs
- `telemetry/gelf` to format the logs

The configuration you need to add is this, and explained below:

```json
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
          "stdout": false
      }
    }
}
```

The GELF configuration parameters for `telemetry/gelf` are:

{{< schema version="v2.3" data="telemetry/gelf.json" >}}

In addition, you must also add the `telemetry/logging`:

{{< schema version="v2.3" data="telemetry/logging.json" filter="level,prefix,stdout,syslog">}}
