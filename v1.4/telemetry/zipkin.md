---
lastmod: 2019-09-15
old_version: true
date: 2019-09-15
notoc: true
linktitle: Zipkin
title: Exporting traces to Zipkin
weight: 90
notoc: true
menu:
  community_v1.4:
    parent: "080 Telemetry"
meta:
  since: v0.5
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - github_com/devopsfaith/krakend-opencensus
  scope:
  - service
---
[Zipkin](https://zipkin.io/) is a distributed tracing system. It helps gather timing data needed to troubleshoot latency problems in service architectures.

The Opencensus exporter allows you export data to Zipkin. Enabling it only requires you to add the `zipkin` exporter in the [opencensus module](/docs/v1.4/telemetry/opencensus/).

The following configuration snippet sends data to your Zipkin:

	"github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
        "zipkin": {
			"collector_url": "http://192.168.99.100:9411/api/v2/spans",
            "service_name": "krakend"
		},
	  }
	}

- `collector_url` is the URL (including port and path) where your Zipkin is accepting the spans
- `service_name` the service name registered in Zipkin


See also the [additional settings](/docs/v1.4/telemetry/opencensus/) of the Opencensus module that can be declared.
