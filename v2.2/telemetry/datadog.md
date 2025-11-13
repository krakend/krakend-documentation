---
lastmod: 2023-02-28
old_version: true
date: 2020-07-24
notoc: true
linktitle: Datadog
title: Exporting traces to Datadog
weight: 90
since: 1.2
source: https://github.com/krakend/krakend-opencensus
images: ["/images/documentation/datadog-screenshot.png"]
menu:
  community_v2.2:
    parent: "080 Telemetry and Analytics"
meta:
  since: v1.2
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
[Datadog](https://www.datadoghq.com/) is a cloud monitoring and security platform for developers, IT operations teams, and businesses.

## Datadog configuration
The Opencensus exporter allows you to export data to Datadog. Enabling it only requires adding the `datadog` exporter in the [opencensus module](/docs/v2.2/telemetry/opencensus/).

The following configuration snippet sends data to your Datadog:
```json
{
      "version": 3,
      "extra_config": {
        "telemetry/opencensus": {
          "sample_rate": 100,
          "reporting_period": 0,
          "exporters": {
            "datadog": {
              "tags": [
                "gw"
              ],
              "global_tags": {
                "env": "prod"
              },
              "disable_count_per_buckets": true,
              "trace_address": "localhost:8126",
              "stats_address": "localhost:8125",
              "namespace": "krakend",
              "service": "gateway"
            }
          }
        }
      }
}
```
As with all [OpenCensus exporters](/docs/v2.2/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema version="v2.2" data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `datadog` entry with the following properties:

{{< schema version="v2.2" data="telemetry/opencensus.json" property="exporters" filter="datadog" >}}

## B3 propagation
The Opencensus module uses B3-style propagation headers, while the rest of your services might be using Datadog-specific propagation headers. If this difference is actual, krakend traces will show up in Datadog, but they won't be connected to the frontend and backend traces.

The `ddtrace-run` adds an option to support B3 style propagation using the environment variables `DD_TRACE_PROPAGATION_STYLE_EXTRACT` and `DD_TRACE_PROPAGATION_STYLE_INJECT`. Use these variables to have your traces perfectly aligned.

For more information, see [its configuration](https://ddtrace.readthedocs.io/en/stable/configuration.html).

## Datadog agent
You must set your Datadog API key in the agent. The exporter communicates with the agent and is the agent the one reporting to Datadog.

Here's an example of how to run the Datadog agent together with KrakenD in a docker compose file:

```yml
krakend:
  image:
ddagent:
  image: gcr.io/datadoghq/agent:latest
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - /proc/:/host/proc/:ro
    - /sys/fs/cgroup/:/host/sys/fs/cgroup:ro
  ports:
    - 8126:8126/tcp
    - 8125:8125/udp
  environment:
    - DD_API_KEY=<API-KEY>
    - DD_APM_ENABLED=true
    - DD_SITE=<DATADOG-SITE>
    - DD_APM_NON_LOCAL_TRAFFIC=true
```

And the configuration would contain:

```json
{ "datadog":
  {
    "@comment": "Rest of the necessary fields intentionally omitted",
    "trace_address": "ddagent:8126",
    "stats_address": "ddagent:8125"
  }
}
```

Notice that we are naming the service `ddagent` in Docker compose, and this is what we have added in the address.