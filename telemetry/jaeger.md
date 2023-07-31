---
lastmod: 2023-02-03
date: 2019-10-11
notoc: true
linktitle: Jaeger
title: Exporting traces to Jaeger
description: The KrakenD exporter to Jaeger allows you to submit spans to a Jaeger Collector (HTTP) or Jaeger Agent (UDP) automatically.
weight: 100
notoc: false
aliases: ["/docs/logging-metrics-tracing/jaeger/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---
The KrakenD exporter to [Jaeger](https://www.jaegertracing.io/) allows you to submit spans to a Jaeger Collector (HTTP) or Jaeger Agent (UDP) automatically.

Jaeger is an open-source, end-to-end distributed tracing system that allows you to monitor and troubleshoot transactions in complex distributed systems. Use Jaeger when you want to see the complete flow of a user request through KrakenD and its connected services.

The Opencensus Jaeger exporter allows you to export spans to Jaeger. Enabling it only requires you to add the `jaeger` exporter in the [opencensus module](/docs/telemetry/opencensus/). You can post spans using two different approaches:

- **Submit spans to an Agent** ([Thrift over UDP](https://www.jaegertracing.io/docs/1.23/apis/#thrift-over-udp-stable)) - using `agent_endpoint`
- **Submit spans to a Collector** ([Thrift over HTTP](https://www.jaegertracing.io/docs/1.23/apis/#thrift-over-http-stable)) - using `endpoint`

You can test this setup by running the **All in One** official Jaeger image and opening the necessary ports. For instance, to run a Jaeger container for UDP and HTTP support do:

{{< terminal title="Term" >}}
docker run -d --name jaeger -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
-p 5775:5775/udp -p 6831:6831/udp -p 6832:6832/udp \
-p 5778:5778 -p 16686:16686 -p 14268:14268 -p 9411:9411 \
jaegertracing/all-in-one:latest
{{< /terminal >}}

Then use `agent_endpoint` pointing to `jaeger:6831` for the Jaeger Agent (UDP) or `endpoint` pointing to `http://jaeger:14268` for the Jaeger Collector (HTTP).

## Collector - Thrift over HTTP
When you don't have a Jaeger Agent deployed next to the application, you can submit all spans to a **Jaeger Collector** over HTTP/HTTPS somewhere else. To do that, use the `endpoint` configuration option inside `jaeger`, as follows:

```json
{
  "extra_config":{
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 0,
      "exporters": {
        "jaeger": {
          "endpoint": "http://jaeger:14268/api/traces",
          "service_name":"krakend",
          "buffer_max_count": 1000
        }
      }
    }
  }
}
```

You can find a running demo with Jaeger using docker-compose on [KrakenD Playground](/docs/overview/playground/).

## Agent - Thrift over UDP
When you want to send spans to a **Jaeger Agent** locally over UDP in Thrift format, you need to use the `agent_endpoint` configuration. To configure the gateway to work with an agent, you will need the following:

```json
{
  "extra_config":{
    "telemetry/opencensus": {
      "sample_rate": 100,
      "reporting_period": 0,
      "exporters": {
        "jaeger": {
          "agent_endpoint": "jaeger:6831",
          "service_name":"krakend",
          "buffer_max_count": 1000
        }
      }
    }
  }
}
```

## Jaeger configuration options
As with all [OpenCensus exporters](/docs/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain the `jaeger` entry with the following properties:

{{< schema data="telemetry/opencensus.json" property="exporters" filter="jaeger" >}}
