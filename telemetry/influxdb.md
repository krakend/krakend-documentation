---
lastmod: 2021-04-28
date: 2019-09-15
notoc: true
linktitle: InfluxDB
title: Exporting metrics and events to InfluxDB
weight: 80
notoc: true
aliases: ["/docs/logging-metrics-tracing/influxdb/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/devopsfaith/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
---
[InfluxDB](https://www.influxdata.com/) is a time series database designed to handle high write and query loads.

The Opencensus exporter allows you export data to [InfluxDB](https://www.influxdata.com) for monitoring metrics and events. Enabling it only requires you to add the `influxdb` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your InfluxDB:

    "telemetry/opencensus": {
      "exporters": {
        "influxdb": {
            "address": "http://192.168.99.100:8086",
            "db": "krakend",
            "timeout": "1s",
            "username": "your-influxdb-user",
            "password": "your-influxdb-password"
        },
      }
    }

- `address` is the URL (including port) where your InfluxDB is installed.
- `db` is the database name.
- `timeout` is the maximum time you will wait for InfluxDB to respond.
- `username` and `password` are optional and used to authenticate against InfluxDB.

See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.
