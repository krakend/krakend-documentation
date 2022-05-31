---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Logstash
title: Logstash
weight: 50
source: https://github.com/krakendio/krakend-logstash
aliases: ["/docs/logging-metrics-tracing/logstash/"]
menu:
  community_current:
    parent: "090 Logging"
---
The [Logstash](https://www.elastic.co/es/logstash/) integration prints **KrakenD logs in JSON format** to ingest them and process them later. If you want to log using the Logstash standard via stdout, you need to add the `telemetry/logging` integration as a dependency.

## Configuration
The configuration you need to enable logstash is:

{{< highlight json >}}
{
    "version": 3,
    "extra_config": {
        "telemetry/logstash": {
            "enabled": true
        },
        "telemetry/logging": {
            "level": "INFO",
            "prefix": "[KRAKEND]",
            "syslog": false,
            "stdout": true,
            "format": "logstash"
        }
    }
}
{{< /highlight >}}
