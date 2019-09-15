---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Logger
title: Exporting to the logger
weight: 130
source: https://github.com/devopsfaith/krakend-opencensus
notoc: true
menu:
  documentation:
    parent: logging-metrics-tracing
---
Opencensus can export data to the system logger as another exporter.

Enabling it only requires you to add the `logger` exporter in the [opencensus module](/docs/logging-metrics-tracing/opencensus/).

The following configuration snippet enables the logger:

	"github_com/devopsfaith/krakend-opencensus": {
      "exporters": {
         "logger": {
            "stats": true,
            "spans": true
        }
	    }
	}

- `stats`: Whether to log the statistics or not
- `spans`: Wether to log the spans or not


See also the [additional settings](/docs/logging-metrics-tracing/opencensus/) of the Opencensus module that can be declared.
