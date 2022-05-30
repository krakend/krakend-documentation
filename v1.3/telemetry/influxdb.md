---
lastmod: 2021-04-28
old_version: true
date: 2019-09-15
notoc: true
linktitle: InfluxDB
title: Exporting metrics and events to InfluxDB
weight: 80
notoc: true
menu:
  community_v1.3:
    parent: "080 Telemetry"
meta:
  since: 0.5
  source: https://github.com/krakendio/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
[InfluxDB](https://www.influxdata.com/) is a time series database designed to handle high write and query loads.

The Opencensus exporter allows you export data to [InfluxDB](https://www.influxdata.com) for monitoring metrics and events. Enabling it only requires you to add the `influxdb` exporter in the [opencensus module](/docs/v1.3/telemetry/opencensus/).

The following configuration snippet sends data to your InfluxDB:

    "github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
        "influxdb": {
            "address": "http://192.168.99.100:8086",
            "db": "krakend",
            "timeout": "1s",
            "username": "your-influxdb-user",
            "password": "your-influxdb-password"
        }
      }
    }

- `address` is the URL (including port) where your InfluxDB is installed.
- `db` is the database name.
- `timeout` is the maximum time you will wait for InfluxDB to respond.
- `username` and `password` are optional and used to authenticate against InfluxDB.

See also the [additional settings](/docs/v1.3/telemetry/opencensus/) of the Opencensus module that can be declared.
