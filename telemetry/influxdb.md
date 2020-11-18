---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: InfluxDB
title: Exporting metrics and events to InfluxDB
weight: 80
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
aliases:
- /docs/logging-metrics-tracing/influxdb/
menu:
  documentation:
    parent: telemetry
---
[InfluxDB](https://www.influxdata.com/) is a time series database designed to handle high write and query loads.

The Opencensus exporter allows you export data to [InfluxDB](https://www.influxdata.com) for monitoring metrics and events. Enabling it only requires you to add the `influxdb` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your InfluxDB:

    "github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
        "influxdb": {
            "address": "http://192.168.99.100:8086",
            "db": "krakend",
            "timeout": "1s"
        },
      }
    }

- `address` is the URL (including port) where your InfluxDB is installed
- `db` is the database name
- `timeout` is the maximum time you wait for Influx to respond

See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.