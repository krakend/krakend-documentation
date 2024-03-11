---
lastmod: 2021-05-02
date: 2018-11-05
linktitle: Metrics API
title: Extended Metrics API
description: Explore the extended metrics available in KrakenD API Gateway telemetry for detailed insights into API performance and usage
aliases: ["/docs/extended-metrics/metrics/"]
weight: 1000
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
notoc: false
meta:
  since: 0.4
  source: https://github.com/krakend/krakend-metrics
  namespace:
  - telemetry/metrics
  scope:
  - service
  log_prefix:
  - "[SERVICE: Stats]"
---
The **metrics API** offers a new `/__stats/` endpoint in a different port and contains a lot of metrics that you can scrap in a custom collector, or you can push them to [InfluxDB](/docs/telemetry/influxdb/).

This component is unrelated to the [OpenTelemetry](/docs/telemetry/opentelemetry/) metrics, and they can coexist. Previous to the creation of OpenTelemetry, the combination of Influx and the metrics API, offered the older versions of [Grafana dashboard](/docs/telemetry/grafana/).

## Configuration
In order to add the metrics API to your KrakenD installation add the `telemetry/metrics` namespace under `extra_config` in the root of your configuration file, e.g.:

{{< highlight go "hl_lines=3-11" >}}
{
  "version": 3,
  "extra_config": {
    "telemetry/metrics": {
      "collection_time": "60s",
      "proxy_disabled": false,
      "router_disabled": false,
      "backend_disabled": false,
      "endpoint_disabled": false,
      "listen_address": ":8090"
    }
  }
}
{{< /highlight >}}

{{< schema data="telemetry/metrics.json" >}}

The structure of the metrics looks like this (truncated):

{{< terminal title="Sample of /__stats endpoint" >}}
curl http://localhost:8090/__stats
{
  "cmdline": [
    "/usr/bin/krakend",
    "run",
    "-c",
    "/etc/krakend/krakend.json",
    "-d"
  ],
  "krakend.router.connected": 0,
  "krakend.router.connected-gauge": 0,
  "krakend.router.connected-total": 0,
  "krakend.router.disconnected": 0,
  "krakend.router.disconnected-gauge": 0,
  "krakend.router.disconnected-total": 0,
  "krakend.service.debug.GCStats.LastGC": 1605724147216402400,
  "krakend.service.debug.GCStats.NumGC": 102,
  "krakend.service.debug.GCStats.Pause.50-percentile": 0,
  "krakend.service.debug.GCStats.Pause.75-percentile": 0,
  "krakend.service.debug.GCStats.Pause.95-percentile": 0,
  "krakend.service.debug.GCStats.Pause.99-percentile": 0,
  "krakend.service.debug.GCStats.Pause.999-percentile": 0,
  "krakend.service.debug.GCStats.Pause.count": 0,
  "krakend.service.debug.GCStats.Pause.max": 0,
  "krakend.service.debug.GCStats.Pause.mean": 0,
  "krakend.service.debug.GCStats.Pause.min": 0,
  "krakend.service.debug.GCStats.Pause.std-dev": 0,
    ...
  }
}
{{< /terminal >}}

## Push metrics to InfluxDB
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
The properties of the `telemetry/influx` are as follows:

{{< schema version="v2.5" data="telemetry/influx.json" >}}

See below how to configure InfluxDB, and you are ready to [publish a Grafana dashboard](/docs/v2.5/telemetry/grafana/).

## Setting up Influx
For **InfluxDB v2.x**, we have included in our [Telemetry Dashboards](https://github.com/krakend/telemetry-dashboards/) the files that create the authorization part.

For **InfluxDB v1.x** (older) the process is straightforward and requires you nothing else than start an Influx instance with the desired configuration.

### Influx v2
If you use Docker, you can start InfluxDB as part of a docker-compose file. You need to specify in the configuration above the same data you used to run InfluxDB. For instance, the following `docker-compose.yml` sets the credentials you need to reflect in the KrakenD configuration.

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
    image: {{< product image >}}:2.5
    volumes:
      - ./krakend:/etc/krakend
    ports:
      - "8080:8080"
```

In the fields `db`, `username`, and `password` of the component configuration reflect the same values as in `DOCKER_INFLUXDB_INIT_BUCKET`, `DOCKER_INFLUXDB_INIT_USERNAME`, and `DOCKER_INFLUXDB_INIT_PASSWORD` accordingly.

The Influx **volume** below must have the contents of the [influx initdb script](https://github.com/krakend/telemetry-dashboards/tree/main/influx), that it will create the authorization needed to let KrakenD push the metrics.

#### Manual configuration
If you don't want to use the automated docker-compose above, the manual steps to create the auth are:

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
When using InfluxDB v1.x, you need to specify in the configuration above the same data you used to run InfluxDB. For instance, the following docker-compose sets the credentials you need to reflect in the KrakenD configuration.

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
    image: {{< product image >}}:2.5
    volumes:
      - ./krakend:/etc/krakend
    ports:
      - "8080:8080"
```

In the fields `db`, `username`, and `password` of the component configuration reflect the same values as in `INFLUXDB_DB`, `INFLUXDB_USER`, and `INFLUXDB_USER_PASSWORD` accordingly.