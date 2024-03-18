---
lastmod: 2021-05-03
old_version: true
date: 2020-11-16
canonical: "/docs/v2.5/telemetry/ocagent/"
notoc: true
linktitle: OpenCensus Agent
title: Exporting metrics, logs, and events to the OpenCensus Agent
weight: 140
source: https://github.com/krakend/krakend-opencensus
notoc: true
menu:
  community_v1.3:
    parent: "080 Telemetry"
meta:
  since: 1.1
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---

The `ocagent` exporter sends OpenCensus Stats and Traces to the OpenCensus Agent, instead of pushing data to backends’ exporters.

For instance, you can enable ocagent to upload data to the OpenCensus Agent, and from there, the agent is simply scraped by a Prometheus.

You can integrate the OpenCensus Agent with Azure Monitor, Jaeger, or Prometheus to name a few examples.

Enabling it only requires you to add the `ocagent` exporter in the [opencensus module](/docs/v1.3/telemetry/opencensus/).

The following configuration snippet sends the data:

    "github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
       "ocagent": {
          "address": "collector",
          "service_name": "krakend",
          "reconnection": "2s",
           "insecure": false,
          "enable_compression": true,
          "headers": {
            "header1": "value1"
          }
        }
      }
    }

- `address` (*string*): The address of your Azure Monitor collector.
- `service_name` (*string*): An identifier of your service.
- `reconnection` (*time*): The reconnection time, you need to specify its units, examples: `1000ms`, `2s`, `1m`
- `insecure` (*bool*): Whether the connection can be established in plain or not.
- `enable_compression` (*bool*): Whether to send data compressed or not.
- `headers` (*object*): List of keys and values for the headers sent:
  - header key: *string*
  - header value: *string*

See also the [additional settings](/docs/v1.3/telemetry/opencensus/) of the Opencensus module that can be declared.
