---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Jaeger
title: Exporting traces to Jaeger
weight: 100
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
aliases:
- /docs/logging-metrics-tracing/jaeger/
menu:
  documentation:
    parent: telemetry
---
[Jaeger](https://www.jaegertracing.io/) is an open source, end-to-end distributed tracing system that allows you to monitor and troubleshoot transactions in complex distributed systems.

The Opencensus exporter allows you export data to Jaeger. Enabling it only requires you to add the `jaeger` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your Jaeger:

	"github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
        "jaeger": {
			"endpoint": "http://192.168.99.100:14268/api/traces",
            "service_name":"krakend"
		},
	  }
	}

- `endpoint` is the URL (including port) where your Jaeger is
- `service_name` the service name registered in Jaeger


See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.
