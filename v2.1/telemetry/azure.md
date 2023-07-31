---
lastmod: 2022-12-07
old_version: true
date: 2020-11-16
linktitle: Azure Monitor
title: Exporting traces to Azure Monitor
weight: 130
menu:
  community_v2.1:
    parent: "080 Telemetry and Analytics"
meta:
  since: 1.1
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
skip_header_image: true
images:
- /images/documentation/screenshots/azure-app-insights-1.png
- /images/documentation/screenshots/azure-app-insights-2.png
- /images/documentation/screenshots/azure-app-insights-3.png
- /images/documentation/diagrams/azure-collector.mmd.png
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


The gateway sends all the traces to a local **OpenTelemetry Collector** ([see repository](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter/azuremonitorexporter)), allowing the gateway to offload data quickly and the collector can take care of additional handling like retries, batching, encryption or even sensitive data filtering. Finally, the Otel Collector pushes all the data to your **Application Insights** on Azure Monitor.

![Otel collector](/images/documentation/diagrams/azure-collector.mmd.png)

## Application Insights configuration
The [OpenCensus exporter](/docs/v2.1/telemetry/opencensus/) is the (stable) component that allows you to export telemetry data, Azure included. Nevertheless, the official **OpenTelemetry Collector** is flagged as **beta** and still **does not support pushing metrics** for Azure. So without further development on the OpenTelemetry side, the integration is limited to traces.

There are three things you need to do:

1) Create the Application Insights resource on Azure
2) Start the OpenTelemetry Collector
3) Add it to KrakenD's configuration

### Create an Application Insights resource
To enable the Azure Monitor integration, you need to add a new resource **Application Insights**, under your Azure account and fill in the information as required (the details provided during the registration form are not relevant to the configuration).

![Application Insights](/images/documentation/screenshots/azure-application-insights.png)

When the resource finishes creating, **save the Instrumentation Key** for later usage, you will find it as shown in the screenshot below:

![Application Insights](/images/documentation/screenshots/azure-application-insights-instrumentation-key.png)

### Starting the OpenTelemetry Collector
You can start the OpenTelemetry Collector with Azure's compatibility using its [Docker image](https://hub.docker.com/r/otel/opentelemetry-collector-contrib). Here there is a `docker-compose.yml` example that includes a KrakenD and a collector:

```yaml
version: "3"
services:
  krakend:
    image: {{< product image >}}:v2.1
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
      - "55678"
```

The configuration mounted for the collector is below, `otel-collector.yaml`:

```yaml
receivers:
  opencensus:

exporters:
  # logging:
  #   verbosity: detailed
  azuremonitor:
    instrumentation_key: XXXXXXX

service:
  # telemetry:
  #   logs:
  #     level: "debug"
  pipelines:
    traces:
      receivers: [opencensus]
      exporters: [azuremonitor]
```
Enable the logging only if you find problems and want extra information.

Replace the value of the `instrumentation_key` entry with the one you got in your Azure dashboard.

### Configuration for KrakenD
Lastly, add at the service level of KrakenD the following configuration:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/opencensus": {
          "sample_rate": 100,
          "reporting_period": 60,
          "exporters": {
              "ocagent": {
                "address": "otel-collector:55678",
                "service_name": "myKrakenD"
              }
          }
      }
  }
}
```
The `otel-collector` above is the name of the Docker-compose service running the collector. You might need to replace it if you are not using this example.

With these three steps, you can start sending data to KrakenD. You should start seeing the graphs populated on Azure Monitor in a couple of minutes.

## Other Resources

- [OpenTelemetry Collector Docker Image](https://hub.docker.com/r/otel/opentelemetry-collector-contrib) with Azure support.
- [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview?tabs=net)
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)