---
lastmod: 2022-10-24
date: 2020-11-16
linktitle: Azure Monitor
title: Exporting metrics, logs and events to Azure Monitor
weight: 130
notoc: true
menu:
  community_current:
    parent: "080 Telemetry and Analytics"
meta:
  since: 1.1
  source: https://github.com/krakendio/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
  log_prefix:
  - "[SERVICE: Opencensus]"
---

[Azure Monitor](https://azure.microsoft.com/en-us/services/monitor/) collect, analyzes, and acts on telemetry data from your Azure and on-premises environments. Azure Monitor helps you maximize performance and availability of your applications and proactively identify problems in seconds.

The Opencensus exporter allows you export data to Azure Monitor. To enable it [add the `ocagent` exporter](/docs/telemetry/ocagent/).