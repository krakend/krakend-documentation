---
lastmod: 2026-11-09
date: 2020-11-16
linktitle: Azure Monitor
title: Azure Monitor Telemetry Integration
description: Integrate Azure Telemetry with KrakenD for effective monitoring and performance analysis of your APIs. Follow our documentation to set up seamless integration with your Azure services.
weight: 140
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  since: v1.1
  source: https://github.com/krakend/krakend-otel
  namespace:
  - telemetry/opentelemetry
  scope:
  - service
  log_prefix:
  - "[SERVICE: OpenTelemetry]"
skip_header_image: true
images:
- /images/documentation/screenshots/azure-app-insights-1.png
- /images/documentation/screenshots/azure-app-insights-2.png
- /images/documentation/screenshots/azure-app-insights-3.png
- /images/documentation/diagrams/azure-collector.mmd.svg
- /images/documentation/screenshots/azure-application-insights-instrumentation-key.png
---

[Azure Monitor](https://azure.microsoft.com/en-us/services/monitor/) collects, analyzes, and acts on telemetry data from your Azure and on-premises environments. Azure Monitor helps you maximize performance and availability of your applications and proactively identify problems in seconds.

<section class="overflow-hidden text-gray-700 ">
  <div class="container">
    <div class="flex flex-wrap -m-1 md:-m-2">
      <div class="flex flex-wrap w-1/3">
        <div class="w-full p-1 md:p-2">
          <a href="/images/documentation/screenshots/azure-app-insights-1.png">
            <img alt="Azure Monitor screenshot" class="shadow-lg hover:scale-110 block object-cover object-center w-full h-full rounded-lg" src="/images/documentation/screenshots/azure-app-insights-1.png">
          </a>
        </div>
      </div>
      <div class="flex flex-wrap w-1/3">
        <div class="w-full p-1 md:p-2">
          <a href="/images/documentation/screenshots/azure-app-insights-2.png">
            <img alt="Azure Monitor screenshot" class="shadow-lg hover:scale-110 block object-cover object-center w-full h-full rounded-lg" src="/images/documentation/screenshots/azure-app-insights-2.png">
          </a>
        </div>
      </div>
      <div class="flex flex-wrap w-1/3">
        <div class="w-full p-1 md:p-2">
          <a href="/images/documentation/screenshots/azure-app-insights-3.png">
            <img alt="Azure Monitor screenshot" class="shadow-lg hover:scale-110 block object-cover object-center w-full h-full rounded-lg" src="/images/documentation/screenshots/azure-app-insights-3.png">
          </a>
        </div>
      </div>
  </div>
</section>


The gateway sends its traces and metrics to a local **OpenTelemetry Collector** ([see repository](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/azuremonitorexporter)), allowing the gateway to offload data quickly and the collector can take care of additional handling like retries, batching, encryption or even sensitive data filtering. Finally, the Otel Collector pushes all the data to your **Application Insights** on Azure Monitor.

![Otel collector](/images/documentation/diagrams/azure-collector.mmd.svg)

## Application Insights configuration
The Azure Monitor exporter of the **OpenTelemetry Collector** sends traces, metrics, and logs to Application Insights. The exporter is flagged as **beta** for all three signals. KrakenD sends its traces and metrics through it.

There are three things you need to do:

1) Create the Application Insights resource on Azure
2) Start the OpenTelemetry Collector
3) Add it to KrakenD's configuration

### Create an Application Insights resource
To enable the Azure Monitor integration, you need to add a new resource **Application Insights**, under your Azure account and fill in the information as required (the details provided during the registration form are not relevant to the configuration).

![Application Insights](/images/documentation/screenshots/azure-application-insights.png)

When the resource finishes creating, **save the Instrumentation Key** or **Connection String** for later usage.

### Starting the OpenTelemetry Collector
You can start the OpenTelemetry Collector with Azure's compatibility using its [Docker image](https://hub.docker.com/r/otel/opentelemetry-collector-contrib). Here there is a `docker-compose.yml` example that includes a KrakenD and a collector:

```yaml
services:
  krakend:
    image: {{< product image >}}:{{< product latest_version >}}
    command: [ "krakend", "run", "-d", "-c", "/etc/krakend/krakend.json"]
    volumes:
      - ./:/etc/krakend
    ports:
      - 8080:8080
  # Collector
  otel-collector:
    image: otel/opentelemetry-collector-contrib
    command: ["--config=/etc/otel-collector.yaml"]
    volumes:
      - ./otel-collector.yaml:/etc/otel-collector.yaml
    ports:
      - "4317"
      - "4318"
```

The configuration mounted for the collector is below, `otel-collector.yaml`:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: "0.0.0.0:4317"
      http:
        endpoint: "0.0.0.0:4318"

exporters:
  # debug:
  #   verbosity: detailed
  azure_monitor:
    connection_string: "InstrumentationKey=XXXXXXX;IngestionEndpoint=XXXXXXX"

service:
  telemetry:
    logs:
      level: "warn"
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [azure_monitor]
    metrics:
      receivers: [otlp]
      exporters: [azure_monitor]
```
Enable the `debug` exporter only if you find problems and want extra information, and add it to the `exporters` list of the pipeline you want to inspect.

Replace the value of the `connection_string` entry with the one you got in your Azure dashboard.

KrakenD exports metrics by default, so keep the `metrics` pipeline: when the collector has none, KrakenD logs a `failed to upload metrics` error on every reporting period. If you only want traces in Azure, remove the `metrics` pipeline and set `"disable_metrics": true` in the KrakenD exporter.

{{< note title="Exporter name on older collectors" type="info" >}}
The exporter has been called `azure_monitor` since August 2026. Collectors released before the rename only know it as `azuremonitor`, and newer collectors still accept that name with a deprecation warning. If your collector rejects `azure_monitor`, use `azuremonitor` in both the `exporters` section and the pipelines.
{{< /note >}}

### Configuration for KrakenD
Lastly, add at the service level of KrakenD the following configuration:

```json
{
  "version": 4,
  "extra_config": {
    "telemetry/opentelemetry": {
      "trace_sample_rate": 1,
      "exporters": {
        "otlp": [
          {
            "name": "azure_monitor",
            "host": "otel-collector",
            "port": 4317
          }
        ]
      }
    }
  }
}
```
The configuration above reports a 100% of the activity (`trace_sample_rate = 1`), when in production consider lowering this value (e.g., `0.1`) as you will want a sample of the traffic only.

The `otel-collector` above is the name of the docker compose service running the collector. You might need to replace it if you are not using this example.

With these three steps, KrakenD starts sending data to Azure Monitor. You should start seeing the graphs populated on Azure Monitor in a couple of minutes.


## Other Resources

- [OpenTelemetry Collector Docker Image](https://hub.docker.com/r/otel/opentelemetry-collector-contrib) with Azure support.
- [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview?tabs=net)
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)