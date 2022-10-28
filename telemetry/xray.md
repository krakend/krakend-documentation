---
lastmod: 2022-10-24
date: 2019-09-15
notoc: true
linktitle: AWS X-Ray
title: Exporting traces to AWS X-Ray
weight: 110
notoc: true
aliases: ["/docs/logging-metrics-tracing/xray/"]
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  source: https://github.com/krakendio/krakend-opencensus
  namespace:
  - telemetry/opencensus
  log_prefix:
  - "[SERVICE: Opencensus]"
  scope:
  - service
---
[AWS X-Ray](https://aws.amazon.com/xray/) is a service offered by Amazon that provides an end-to-end view of requests as they travel through your application, and shows a map of your application’s underlying components.

The Opencensus exporter allows you export data to AWS X-Ray. Enabling it only requires you to add the `xray` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your X-Ray:

```json
{
  "extra_config": {
    "telemetry/opencensus": {
      "exporters": {
        "xray": {
          "version": "latest",
          "region": "eu-west-1",
          "use_env": false,
          "access_key_id": "myaccesskey",
          "secret_access_key": "mysecretkey"
        }
      }
    }
  }
}
```
As with all [OpenCensus exporters](/docs/telemetry/opencensus/), you can add optional settings in the `telemetry/opencensus` level:

{{< schema data="telemetry/opencensus.json" filter="sample_rate,reporting_period,enabled_layers">}}

Then, the `exporters` key must contain an `xray` entry with the following properties:

{{< schema data="telemetry/opencensus.json" property="exporters" filter="xray" >}}

See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.
