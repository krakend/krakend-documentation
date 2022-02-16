---
lastmod: 2021-04-28
date: 2020-11-18
linktitle: InfluxDB (native)
title: Native InfluxDB exporter
weight: 20
notoc: true
aliases: ["/docs/extended-metrics/influxdb/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/devopsfaith/krakend-influx
  namespace:
  - telemetry/influx
  - telemetry/metrics
  scope:
  - service
  log_prefix:
  - "[SERVICE: InfluxDB]"
---

KrakenD can expose very detailed metrics to provide a **monitoring dashboard**. One of the richest monitoring solutions at the metrics level is the combination of [krakend-metrics](/docs/telemetry/extended-metrics/) with the native **krakend-influx** exporter. The two components let you send detailed metrics to InfluxDB and draw them later on our preconfigured [Grafana dashboard](/docs/telemetry/grafana/) can feed from here and provide you a useful.

Notice that there are **two different implementations** of InfluxDB in KrakenD:

- Native InfluxDB exporter (this page)
- [OpenCensus InfluxDB exporter](/docs/telemetry/influxdb/)

{{< note title="Which InfluxDB implementation should I choose?" >}}
The **native implementation** exports data from a collector that is tailor-made for KrakenD, and is **richer in content** and less abstract. On the other hand, the [OpenCensus exporter for InfluxDB](/docs/telemetry/influxdb/) is more generalistic and abstract, but implements a collector with less data. For our Grafana dahsboard, choose the native one.
{{< /note >}}

## InfluxDB configuration

Pushing data to InfluxDB requires adding two different configuration pieces:

- Enabling the extended metrics (collecting the information)
- Enabling InfluxDB (pushing the data)

You can accomplish it with the following snippet.

{{< highlight json >}}
{
    "version": 3,
    "extra_config": {
      "telemetry/influx":{
          "address":"http://192.168.99.9:8086",
          "ttl":"25s",
          "buffer_size":0,
          "db": "krakend",
          "username": "your-influxdb-user",
          "password": "your-influxdb-password"
      },
      "telemetry/metrics": {
        "collection_time": "30s",
        "listen_address": "127.0.0.1:8090"
      }
    }
}
{{< /highlight >}}


- `address` (*string*): The complete url of the influxdb including the port if different from defaults in http/https
- `ttl` (*duration*): Valid time units are: `ns` (nanoseconds), `us` or `µs` (microseconds), `ms` (milliseconds), `s` (seconds), `m` (minutes - don't!), `h` (hours - don't!)
- `buffer_size` (*integer*): Use `0` to send events immediately or set the number of points that should be sent together.
- `db` (*string*): Name of the database, defaults to *krakend*.
- `username` and `password` are optional and used to authenticate against InfluxDB.

Now you are ready to [publish a Grafana dashboard](/docs/telemetry/grafana/).
