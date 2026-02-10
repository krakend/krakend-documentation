---
lastmod: 2022-10-25
old_version: true
date: 2019-09-15
notoc: true
linktitle: Logging to Graylog - GELF
title: Graylog GELF Logging Integration
description: Integrate Graylog GELF logging with KrakenD API Gateway for centralized log management and analysis of API logs
weight: 320
menu:
  community_v2.12:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: v0.7
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

{{< schema version="v2.12" data="telemetry/gelf.json" >}}

In addition, you must also add the `telemetry/logging`:

{{< schema version="v2.12" data="telemetry/logging.json" filter="level,prefix,stdout,syslog">}}
