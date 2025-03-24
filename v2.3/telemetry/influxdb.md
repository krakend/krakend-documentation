---
lastmod: 2022-10-24
old_version: true
date: 2019-09-15
linktitle: InfluxDB
title: Exporting metrics and events to InfluxDB
weight: 10
#notoc: true
menu:
  community_v2.3:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/krakend/krakend-influx
  namespace:
  - telemetry/opencensus
  - telemetry/influx
  - telemetry/metrics
  scope:
  - service
  log_prefix:
  - "[SERVICE: InfluxDB]"
  - "[SERVICE: Opencensus]"
---
KrakenD can expose very detailed metrics to provide a **monitoring dashboard**. One of the richest monitoring solutions at the metrics level is the combination of [Extended metrics](/docs/v2.3/telemetry/extended-metrics/) with the **native Influx** exporter. The two components let you send detailed metrics to InfluxDB and draw them later on our preconfigured [Grafana dashboard](/docs/v2.3/telemetry/grafana/).

## InfluxDB configuration
Notice that this document describes **two different implementations** of InfluxDB:

- Native InfluxDB exporter (**recommended**)
- OpenCensus InfluxDB exporter

{{< note title="Which InfluxDB implementation should I choose?" >}}
The **native implementation** exports data from a collector that is tailor-made and maintained by KrakenD and is **richer in content**. On the other hand, the OpenCensus exporter for InfluxDB is more generalistic and abstract and is configured similarly to other systems but implements a collector with fewer data. For our Grafana dashboard, choose the native one.
{{< /note >}}

### Native InfluxDB configuration

Pushing data to InfluxDB using the native middleware requires adding two different configuration pieces:

- Enabling the extended metrics (collecting the information)
- Enabling InfluxDB (pushing the data)

You can accomplish it with the following snippet.

```json
{
    "version": 3,
    "extra_config": {
      "telemetry/influx":{
          "address":"http://influxdb:8086",
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
```

{{< schema version="v2.3" data="telemetry/influx.json" >}}

See below how to configure InfluxDB, and you are ready to [publish a Grafana dashboard](/docs/v2.3/telemetry/grafana/).

### OpenCensus InfluxDB exporter
[InfluxDB](https://www.influxdata.com/) is a time series database designed to handle high write and query loads.

The Opencensus exporter allows you export data to [InfluxDB](https://www.influxdata.com) for monitoring metrics and events. Enabling it only requires you to add as `exporter` the `influxdb` entry.

The following configuration snippet sends data to your InfluxDB:

```json
{
    "telemetry/opencensus": {
      "sample_rate": 100,
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
}
```
As with all [OpenCensus exporters](/docs/v2.3/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.3" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}


Then, the `exporters` key must contain an `influxdb` entry with the following properties:

{{< schema version="v2.3" data="telemetry/opencensus.json" property="exporters" filter="influxdb" >}}


## Setting up Influx
For **InfluxDB v2.x**, we have included in our [Telemetry Dashboards](https://github.com/krakend/telemetry-dashboards/) the files that create the authorization part.

For **InfluxDB v1.x** (older) the process is straightforward and requires you nothing else than start an Influx instance with the desired configuration.

### Influx v2
If you use Docker, you can start InfluxDB as part of a docker compose file. You need to specify in the configuration above the same data you used to run InfluxDB. For instance, the following `docker-compose.yml` sets the credentials you need to reflect in the KrakenD configuration.

```yaml
version: "3"
services:
  influxdb:
    image: influxdb:2.4
    environment:
      - "DOCKER_INFLUXDB_INIT_MODE=setup"
      - "DOCKER_INFLUXDB_INIT_USERNAME=krakend-dev"
      - "DOCKER_INFLUXDB_INIT_PASSWORD=pas5w0rd"
      - "DOCKER_INFLUXDB_INIT_ORG=my-org"
      - "DOCKER_INFLUXDB_INIT_BUCKET=krakend"
      - "DOCKER_INFLUXDB_INIT_RETENTION=1w"
      - "DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=my-super-secret-auth-token"
    ports:
      - "8086:8086"
    volumes:
      - "./config/telemetry/influx:/docker-entrypoint-initdb.d"
  krakend:
    image: {{< product image >}}:2.3
    volumes:
      - ./krakend:/etc/krakend
    ports:
      - "8080:8080"
```

In the fields `db`, `username`, and `password` of the component configuration reflect the same values as in `DOCKER_INFLUXDB_INIT_BUCKET`, `DOCKER_INFLUXDB_INIT_USERNAME`, and `DOCKER_INFLUXDB_INIT_PASSWORD` accordingly.

The Influx **volume** below must have the contents of the [influx initdb script](https://github.com/krakend/telemetry-dashboards/tree/main/influx), that it will create the authorization needed to let KrakenD push the metrics.

#### Manual configuration
If you don't want to use the automated docker compose above, the manual steps to create the auth are:

Connect to your influx
{{< terminal title="Term" >}}
docker exec -it influxdb /bin/bash
{{< /terminal >}}

Create a configuration:

{{< terminal title="Term" >}}
influx config create --config-name krakend-config \
  --host-url http://localhost:8086 \
  --org my-org \
  --token my-super-secret-auth-token \
  --active
{{< /terminal >}}

Make sure to replace the values below as follows:

- `config-name` is a random name to identify your configuration
- `org` Your organization name as written during the setup of InfluxDB
- `token` The one you started influx with
- `host-url` The address where the influxdb is running (inside Docker is as shown)

Access with a web browser to `http://localhost:8086` and select **BUCKETS** from the UI. You'll see that there is an ID next to the bucket you created during the Setup. **Copy the bucket ID**.

![Getting a token](/images/documentation/influx_bucket.png)

And now launch the last command in the shell:

{{< terminal title="Term" >}}
influx v1 auth create \
  --read-bucket b492e6f8f3b13aaa \
  --write-bucket b492e6f8f3b13aaa \
  --username krakend-dev
{{< /terminal >}}

Replace the ID of the buckets above with the ID you just copied, and the username as in the docker compose. The shell will ask for your password.

Now your configuration should work and start sending data to InfluxDB:

```json
{
    "version": 3,
    "extra_config": {
        "telemetry/metrics": {
            "collection_time": "60s",
            "listen_address": ":8090"
        },
        "telemetry/influx": {
            "address": "http://localhost:8086",
            "ttl": "25s",
            "buffer_size": 100,
            "db": "krakend_db",
            "username": "user",
            "password": "password"
        }
    }
}
```

Make sure to type in `db` the bucket name you created on InfluxDB and the `username` and `password` as well.

### Influx v1
When using InfluxDB v1.x, you need to specify in the configuration above the same data you used to run InfluxDB. For instance, the following docker compose sets the credentials you need to reflect in the KrakenD configuration.

```yaml
version: "3"
services:
  influxdb:
    image: influxdb:1.8
    environment:
      - "INFLUXDB_DB=krakend"
      - "INFLUXDB_USER=krakend-dev"
      - "INFLUXDB_USER_PASSWORD=pas5w0rd"
      - "INFLUXDB_ADMIN_USER=admin"
      - "INFLUXDB_ADMIN_PASSWORD=supersecretpassword"
    ports:
      - "8086:8086"
  krakend:
    image: {{< product image >}}:2.3
    volumes:
      - ./krakend:/etc/krakend
    ports:
      - "8080:8080"
```

In the fields `db`, `username`, and `password` of the component configuration reflect the same values as in `INFLUXDB_DB`, `INFLUXDB_USER`, and `INFLUXDB_USER_PASSWORD` accordingly.