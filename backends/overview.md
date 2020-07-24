---
lastmod: 2018-10-21
date: 2016-09-30
toc: true
linktitle: Backends Overview
title: Backends Overview
weight: -1000
menu:
  documentation:
    parent: backends
---

The concept of "backend" references to the origin servers providing the necessary data to populate your endpoints.

A backend can be any server inside our outside your networks as long it is reachable by KrakenD. For instance, you can create endpoints that are fetching data from your internal servers and enrich them by adding third-party data from an external API like Github, Facebook or any other service.

The backends are declared inside every endpoint using the `backend` key.

## Simple example
In the example below, KrakenD offers an endpoint `/v1/products` that when requested connects simultaneously to the two services declared under each `host` field (but picking just one of each using load balancing) and returns the merged content given by `/products/_catalog/all` and `/marketing/offers` of these two different services.

```
...
"endpoints": [
{
	"endpoint": "/v1/products",
	"method": "GET",
	"backend": [
		{
			"url_pattern": "/products/_catalog/all",
			"host": [
				"https://products-01.myapi.com:8000",
				"https://products-02.myapi.com:8000",
				"https://products-03.myapi.com:8000"
			]
		},
		{
			"url_pattern": "/marketing/offers",
			"host": [
				"https://marketing.myapi.com:8000"
			]
		}
	]
},
...
```
