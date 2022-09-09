---
lastmod: 2020-11-17
date: 2020-11-17
linktitle: Grafana Dashboard
title: Preconfigured Grafana dashboard
weight: 20
aliases: ["/docs/extended-metrics/grafana/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
skip_header_image: true
images:
- /images/documentation/grafana-screenshot.png
meta:
  since: 0.5
  source: https://grafana.com/grafana/dashboards/15029-krakend/
  namespace:
  - telemetry/influx
  - telemetry/metrics
  scope:
  - service
---

The preconfigured Grafana dashboard for KrakenD offers valuable information to understand the performance of your services and detect anomalies in the service.

The dashboard is extensive and offers you metrics like:

- Requests from users to KrakenD
- Requests from KrakenD to your backends
- Response times
- Memory usage and details
- Endpoints and status codes
- Heatmaps
- Open connections
- Throughput
- Distributions, timers, garbage collection and a long etcetera

{{< youtube Ik18Zlwyap8 >}}


## Configure Grafana
Add the following configuration to your `krakend.json` at the root level:

{{< highlight json >}}
{
  "version": 3,
  "extra_config": {
    "telemetry/influx":{
        "address":"http://192.168.99.9:8086",
        "ttl":"25s",
        "buffer_size":0
    },
    "telemetry/metrics": {
      "collection_time": "30s",
      "listen_address": "127.0.0.1:8090"
    }
  }
}
{{< /highlight >}}

For more details of this configuration see the [InfluxDb exporter](/docs/telemetry/influxdb/)

Then, import our [Grafana dashboard for Krakend](https://grafana.com/grafana/dashboards/15029-krakend/ ).

## Importing the Grafana dashboard
To import the dashboard: From the Grafana UI, click the + icon in the side menu, and then click Import. Choose import via Grafana.com and use the ID `15029`.

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

4) Go to the browser and open **http://localhost:3000**. Use `admin` for both user and password

5) Find the button to add the Data Source in the home screen. Select InfluxDB as the database and fill the details you provided when starting influxdb:

- URL: `http://localhost:8086`
- Access: `Browser`
- database: `krakend`
- password: `supersecretpassword`
- HTTP Method : `GET`

5) Import the Dashboard via grafana.com. Type `15029` and click on Load. The Dashboard will be ready for you!

![Grafana KrakenD Dashboard](/images/documentation/grafana-screenshot.png)
