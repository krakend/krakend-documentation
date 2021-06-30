---
lastmod: 2021-01-28
old_version: true
date: 2016-09-30
toc: true
linktitle: Declaring backends
title: Backends Overview
weight: -1000
menu:
  community_v1.3:
    parent: "050 Backends Configuration "
---

The concept of `backend` refers to the origin servers providing the necessary data to populate your endpoints. A backend can be something like your HTTP-based API, a Lambda function, or a Kafka queue, to name a few examples.

A backend can be any server inside or outside your network, as long it is reachable by KrakenD. For instance, you can create endpoints fetching data from your internal servers and enrich them by adding third-party data from an external API like Github, Facebook, or any other service, and return back everything aggregated in a single glorified response.

When a KrakenD endpoint is hit, the engine requests **all defined backends in parallel** (unless a [sequential proxy](/docs/v1.3/endpoints/sequential-proxy/) is used). The returned content is parsed according to its `encoding` or middleware configuration.

The backends are declared inside every endpoint using a `backend` array. 

## Backend configuration
Inside the `backend` array, you need to create an object for each backend entry. The more important options are:

- `encoding`: Define your [needed encoding](/docs/v1.3/backends/supported-encodings/) to inform KrakenD how to parse the response.
- `sd`: When you use a [Service Discovery](/docs/v1.3/service-discovery/overview/) system to resolve your backend services (e.g., when deploying in k8s)
- `method`: One of `GET`, `POST`, `PUT`, `DELETE`, `PATCH` (in **uppercase**!). The method does not need to match the endpoint's method.
- `url_pattern` The path inside the service (no protocol, no host, no method). E.g: `/users`
- `host` an array with all the available hosts to load balance requests using the format `protocol://host:port`. E.g.: ' https://my.users-ms.com`.
- `extra_config` when there is additional configuration related to a specific component or middleware that you want to enable

Other configuration options such as the ones for [data manipulation](/docs/v1.3/backends/data-manipulation/) are available. You will find them in each specific feature section. 

### Default values
When declaring a backend, all properties named as the endpoint are automatically inherited. For instance, if your endpoint uses a `POST` `method`, unless otherwise declared, the backend will use `POST` as well.

There are also some attributes that you can omit if you don't need to override the value:

- `host`: If you have a `host` declaration at the configuration's root level, it's value is used.
- `method`: The endpoint's method is used, declare it to alter it in the backend call.
- `encoding`: Will use the endpoint encoding, or `json` if not defined anywhere.
- `sd`: The service discovery always defaults to `static` (no external Service Discovery).
- `extra_config`: Not required unless additional non-core middleware is needed (like a circuit breaker, rate limit, etc.)

## Backend configuration example
In the example below, KrakenD offers an endpoint `/v1/products` that merges the content from two different services using the URLs `/products` and `/offers`. The marketing (`marketing.myapi.com`) and the products (`products-XX.myapi.com`) API requests are fired simultaneously. KrakenD will load balance among the listed hosts (here or in your [service discovery](/docs/v1.3/service-discovery/overview/)) to pick one of the three hosts.

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
                "https://products-02.myapi.com:8000",
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
