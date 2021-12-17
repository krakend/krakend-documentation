---
lastmod: 2021-04-28
old_version: true
date: 2020-11-18
linktitle: InfluxDB exporter
title: Native InfluxDB exporter
weight: 20
notoc: true
menu:
  community_v1.3:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/letgoapp/krakend-influx
  namespace:
  - github_com/letgoapp/krakend-influx
  - github_com/devopsfaith/krakend-metrics
  scope:
  - service
---

KrakenD can expose detailed and extended metrics via the [krakend-metrics](/docs/v1.3/extended-metrics/influxdb/)). The **krakend-influx** component lets you **send these extended KrakenD metrics to InfluxDB**.

Notice that there are **two different implementations** of InfluxDB in KrakenD:

- Native InfluxDB exporter (this page)
- [OpenCensus InfluxDB exporter](/docs/v1.3/telemetry/influxdb/)

{{< note title="Which InfluxDB implementation should I choose?" >}}
The **native implementation** exports data from a collector that is tailor-made for KrakenD, and also richer in content and less abstract. On the other hand, the [OpenCensus exporter for InfluxDB](/docs/v1.3/telemetry/influxdb/) is more generalistic and abstract, but implements a collector with less data. For our Grafana dahsboard, choose this one.
{{< /note >}}

## InfluxDB configuration

Pushing data to InfluxDB requires adding two different configuration pieces:

- Enabling the extended metrics (collecting the information)
- Enabling InfluxDB (pushing the data)

You can accomplish it with the following snippet.

    {
      "version": 2,
      "extra_config": {
        "github_com/letgoapp/krakend-influx":{
            "address":"http://192.168.99.9:8086",
            "ttl":"25s",
            "buffer_size":0,
            "db": "krakend",
            "username": "your-influxdb-user",
            "password": "your-influxdb-password"
        },
        "github_com/devopsfaith/krakend-metrics": {
          "collection_time": "30s",
          "listen_address": "127.0.0.1:8090"
        }
      }
    }

- `address` (*string*): The complete url of the influxdb including the port if different from defaults in http/https
- `ttl` (*duration*): Expressed as <value><units> (e.g: `30s`,`1m`). See [accepted values](https://golang.org/pkg/time/#ParseDuration).
- `buffer_size` (*integer*): Use `0` to send events immediately or set the number of points that should be sent together.
- `db` (*string*): Name of the database, defaults to *krakend*.
- `username` and `password` are optional and used to authenticate against InfluxDB.

Now you are ready to [publish a Grafana dashboard](/docs/v1.3/extended-metrics/grafana/).
