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

The concept of `backend` references to the origin servers providing the necessary data to populate your endpoints. A backend can be something as your HTTP-based API, a Lambda function, or a Kafka queue to name a few examples.

A backend can be any server inside our outside your network, as long it is reachable by KrakenD. For instance, you can create endpoints that are fetching data from your internal servers and enrich them by adding third-party data from an external API like Github, Facebook or any other service.

When a KrakenD endpoint is hit, the engine requests **all defined backends in parallel** (unless a [sequential proxy](/docs/endpoints/sequential-proxy) is used), and the content parsed according to its `encoding` or middleware configuration.

The backends are declared inside every endpoint using a `backend` array. 

## Backend configuration
Inside the `backend` array, you need to create an object for each backend entry. The more important options are:

- [`encoding`](/docs/backends/supported-encodings/) to define how to parse the backend response
- [`sd`](/docs/service-discovery/overview/) when using a Service Discovery system (e.g., when deploying in k8s)
- `method` one of `GET`, `POST`, `PUT`, `DELETE`, `PATCH` (in **uppercase**!)
- `url_pattern` The path inside the service (no protocol, no host, no method)
- `host` an array with all the available hosts to load balance requests using the format `protocol://host:port`
- `extra_config` when there is additional configuration related to a specific component or middleware that you want to enable

More configuration options such as the ones for [data manipulation](/docs/backends/data-manipulation/) are available. You will find them in each specific section. 

## Backend configuration example
In the example below, KrakenD offers an endpoint `/v1/products` that merges the content from two different services using the URLs `/products` and `/offers`. The requests to the marketing (`marketing.myapi.com`) and the products (`products-XX.myapi.com`) API are fired simultaneously. KrakenD will load balance amongs the listed hosts (here or in your [service discovery](/docs/service-discovery/overview/)) to pick one of the three hosts.

```
...
"endpoints": [
{
	"endpoint": "/v1/products",
	"method": "GET",
	"backend": [
		{
			"url_pattern": "/products",
			"host": [
				"https://products-01.myapi.com:8000",
				"https://products-02.myapi.com:8000"
				"https://products-03.myapi.com:8000"
			]
		},
		{
			"url_pattern": "/offers",
			"host": [
				"https://marketing.myapi.com:8000"
			]
		}
	]
},
...
```
