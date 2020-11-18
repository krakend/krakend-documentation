---
lastmod: 2020-11-17
date: 2020-11-17
linktitle: Grafana dashboard
title: Preconfigured Grafana dashboard
weight: 30
notoc: true
menu:
  documentation:
    parent: extended-metrics
images:
- /images/documentation/grafana-screenshot.png
---

The Grafana dashboard for KrakenD offers valuable information to understand the performance of your services and detect anomalies in the service. The dashboard is extensive and offers you metrics like:

- Requests from users to KrakenD
- Requests from KrakenD to your backends
- Response times
- Memory usage and details
- Endpoints and status codes
- Heatmaps
- Open connections
- Throughput
- Distributions, timers, garbage collection and a long etcetera

Better an image than a thousand words (video with no sound):

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ik18Zlwyap8" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Configure Grafana

Add the following configuration to your `krakend.json` at the root level:

    {
      "version": 2,
      "extra_config": {
        "github_com/letgoapp/krakend-influx":{
            "address":"http://192.168.99.9:8086",
            "ttl":"25s",
            "buffer_size":0
        },
        "github_com/devopsfaith/krakend-metrics": {
          "collection_time": "30s",
          "listen_address": "127.0.0.1:8090"
        }
      }
    }

For more details of this configuration see the [InfluxDb exporter](/docs/extended-metrics/influxdb/)

Then, import our [Grafana dashboard for Krakend](https://grafana.com/dashboards/5722).

## Importing the Grafana dashboard

To import the dashboard: From the Grafana UI, click the + icon in the side menu, and then click Import. Choose import via Grafana.com and use the ID `5722`.

## Local testing with Docker
After adding your configuration to KrakenD, to test the configuration locally with Docker, you will need to:

1) Start an InfluxDB:

{{< terminal title="Start InfluxDB" >}}
docker run -p 8086:8086 \
	  -e INFLUXDB_DB=krakend \
	  -e INFLUXDB_USER=letgo -e INFLUXDB_USER_PASSWORD=pas5w0rd \
	  -e INFLUXDB_ADMIN_USER=admin -e INFLUXDB_ADMIN_PASSWORD=supersecretpassword \
	  -it --name=influx \
	  influxdb
{{< /terminal >}}

2) Open the CLI:

{{< terminal title="CLI" >}}
docker exec -it influx influx
{{< /terminal >}}

3) Start Grafana:

{{< terminal title="Grafana" >}}
docker run \
  -d \
  -p 3000:3000 \
  --name=grafana \
  grafana/grafana
{{< /terminal >}}

4) Import the Dashboard