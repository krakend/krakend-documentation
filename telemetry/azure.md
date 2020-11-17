---
lastmod: 2020-11-16
date: 2020-11-16
notoc: true
linktitle: Azure Monitor
title: Exporting metrics, logs and events to Azure Monitor
weight: 130
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
menu:
  documentation:
    parent: telemetry
---

[Azure Monitor](https://azure.microsoft.com/en-us/services/monitor/) collect, analyzes, and acts on telemetry data from your Azure and on-premises environments. Azure Monitor helps you maximize performance and availability of your applications and proactively identify problems in seconds.

The Opencensus exporter allows you export data to Azure Monitor. Enabling it only requires you to add the `ocagent` exporter in the [opencensus module](/docs/telemetry/opencensus/), see [how to configure the OpenCensus Agent](/docs/telemetry/ocagent/).
