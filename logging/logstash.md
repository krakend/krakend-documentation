---
lastmod: 2022-06-15
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
The [Logstash](https://www.elastic.co/es/logstash/) integration prints **KrakenD application logs in JSON format** (not access logs) to ingest them and process them later. If you want to log using the Logstash standard via stdout, you need to add the `telemetry/logging` integration as a dependency.

## Configuration
The configuration you need to enable logstash is:

{{< highlight json >}}
{
    "version": 3,
    "extra_config": {
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

When the `format` of the logging format is `logstash` then the output of the application log (not the access log) is printed in JSON format, as follows:

    {"@timestamp":"2022-06-15T15:37:02.619+00:00", "@version": 1, "level": "DEBUG", "message": "[SERVICE: Gin] Debug enabled", "module": "KRAKEND"}
    {"@timestamp":"2022-06-15T15:37:02.619+00:00", "@version": 1, "level": "INFO", "message": "Starting the KrakenD instance", "module": "KRAKEND"}
    {"@timestamp":"2022-06-15T15:37:02.619+00:00", "@version": 1, "level": "DEBUG", "message": "[ENDPOINT: /test] Building the proxy pipe", "module": "KRAKEND"}
    {"@timestamp":"2022-06-15T15:37:02.619+00:00", "@version": 1, "level": "DEBUG", "message": "[BACKEND: /404] Building the backend pipe", "module": "KRAKEND"}

The application **access log** will still show in plain text. For example, you might see the application logs in JSON and the **application logs** in JSON. For example:

    {"@timestamp":"2022-06-15T15:37:07.619+00:00", "@version": 1, "level": "DEBUG", "message": "[SERVICE: Telemetry] Registering usage stats for Cluster ID GH9V15Rf22Zp3F4HBtvF9NkGO9WL7HKp8h7St7l+qc0=", "module": "KRAKEND"}
    [GIN] 2022/06/15 - 15:38:58 | 200 |       4.529µs |      172.17.0.1 | GET      "/test/access"

If you don't want to parse the access log on Logstash, you can remove it from the stdout using `disable_access_log` (see how to [remove requests from logs options](/docs/service-settings/router-options/#remove-requests-from-logs)).