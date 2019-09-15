---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: InfluxDB
title: Exporting metrics and events to InfluxDB
weight: 80
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
menu:
  documentation:
    parent: logging-metrics-tracing
---
[InfluxDB](https://www.influxdata.com/) is a time series database designed to handle high write and query loads.

The Opencensus exporter allows you export data to [InfluxDB](https://www.influxdata.com) for monitoring metrics and events. Enabling it only requires you to add the `influxdb` exporter in the [opencensus module](/docs/logging-metrics-tracing/opencensus/).

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

See also the [additional settings](/docs/logging-metrics-tracing/opencensus/) of the Opencensus module that can be declared.