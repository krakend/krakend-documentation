---
lastmod: 2020-11-16
old_version: true
date: 2020-11-16
linktitle: Azure Monitor
title: Exporting metrics, logs and events to Azure Monitor
weight: 130
notoc: true
menu:
  community_v1.4:
    parent: "080 Telemetry"
meta:
  since: 1.1
  source: https://github.com/devopsfaith/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---

[Azure Monitor](https://azure.microsoft.com/en-us/services/monitor/) collect, analyzes, and acts on telemetry data from your Azure and on-premises environments. Azure Monitor helps you maximize performance and availability of your applications and proactively identify problems in seconds.

The Opencensus exporter allows you export data to Azure Monitor. Enabling it only requires you to add the `ocagent` exporter in the [opencensus module](/docs/v1.4/telemetry/opencensus/), see [how to configure the OpenCensus Agent](/docs/v1.4/telemetry/ocagent/).
