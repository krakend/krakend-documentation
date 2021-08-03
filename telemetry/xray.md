---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: AWS X-Ray
title: Exporting traces to AWS X-Ray
weight: 110
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
aliases: ["/docs/logging-metrics-tracing/xray/"]
menu:
  community_current:
    parent: "080 Telemetry"
---
[AWS X-Ray](https://aws.amazon.com/xray/) is a service offered by Amazon that provides an end-to-end view of requests as they travel through your application, and shows a map of your application’s underlying components.

The Opencensus exporter allows you export data to AWS X-Ray. Enabling it only requires you to add the `xray` exporter in the [opencensus module](/docs/telemetry/opencensus/).

The following configuration snippet sends data to your X-Ray:

	"github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
        "xray": {
			"version": "latest",
            "region": "eu-west-1",
			"use_env": false,
            "access_key_id": "myaccesskey",
            "secret_access_key": "mysecretkey"
		},
	  }
	}

- `version`: The version of the AWS X-Ray service to use.
- `region`: The AWS geographical region.
- `use_env`: When `true` the AWS credentials (`access_key_id` and `secret_access_key`) are taken from environment vars. Don't specify them then.
- `access_key_id`: Your access key ID provided by Amazon. Needed when `use_env` is unset or set to `false`.
- `secret_access_key`: Your secret access key provided by Amazon. Needed when `use_env` is unset or set to `false`.


See also the [additional settings](/docs/telemetry/opencensus/) of the Opencensus module that can be declared.
