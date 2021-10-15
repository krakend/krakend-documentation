---
lastmod: 2019-10-11
date: 2019-10-11
notoc: true
linktitle: Jaeger
title: Exporting traces to Jaeger
weight: 100
notoc: true
aliases: ["/docs/logging-metrics-tracing/jaeger/"]
menu:
  community_current:
    parent: "080 Telemetry"
meta:
  since: 0.5
  source: https://github.com/devopsfaith/krakend-opencensus
  namespace:
  - telemetry/opencensus
  scope:
  - service
---
[Jaeger](https://www.jaegertracing.io/) is an open source, end-to-end distributed tracing system that allows you to monitor and troubleshoot transactions in complex distributed systems.

The Opencensus exporter allows you export data to Jaeger. Enabling it only requires you to add the `jaeger` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your Jaeger:

	"telemetry/opencensus": {
    "exporters": {
      "jaeger": {
			  "endpoint": "http://192.168.99.100:14268/api/traces",
        "service_name":"krakend",
        "buffer_max_count": 1000
		  },
	  }
	}

- `endpoint` is the URL (including port) where your Jaeger is
- `service_name` the service name registered in Jaeger
- `buffer_max_count` defines the total number of traces that can be buffered in memory


See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.
