---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Jaeger
title: Exporting traces to Jaeger
weight: 100
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
menu:
  documentation:
    parent: logging-metrics-tracing
---
[Jaeger](https://www.jaegertracing.io/) is an open source, end-to-end distributed tracing system that allows you to monitor and troubleshoot transactions in complex distributed systems.

The Opencensus exporter allows you export data to Jaeger. Enabling it only requires you to add the `jaeger` exporter in the [opencensus module](/docs/logging-metrics-tracing/opencensus/).

The following configuration snippet sends data to your Jaeger:

	"github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
        "jaeger": {
			"endpoint": "http://192.168.99.100:14268",
            "service_name":"krakend"
		},
	  }
	}

- `endpoint` is the URL (including port) where your Jaeger is
- `serviceName` the service name registered in Jaeger


See also the [additional settings](/docs/logging-metrics-tracing/opencensus/) of the Opencensus module that can be declared.
