---
lastmod: 2025-01-09
old_version: true
date: 2016-09-30
toc: true
linktitle: The backend object
title: Backend Configuration
description: Explore the backend configuration options in KrakenD API Gateway, allowing you to connect and integrate with your microservices efficiently
weight: 20
notoc: true
menu:
  community_v2.11:
    parent: "040 Routing and Forwarding"
---
The concept of `backend` refers to the origin servers providing the necessary data to populate your endpoints. A backend can be something like your HTTP-based API, a Lambda function, or a Kafka queue, for example.

A `backend` can be any server inside or outside your network if it is reachable by KrakenD. For instance, you can create endpoints fetching data from your internal servers and enrich them by adding third-party data from an external API like Github, Facebook, or other services. You can also return everything aggregated in a single glorified response.

A `backend` object is an array of all the services that an [`endpoint`](/docs/v2.11/endpoints/) connects to. It defines the list of hostnames connected to and the URL to send or receive the data. If a backend has more than one `host`, then the array is the [egress load balancing list](/docs/v2.11/throttling/load-balancing/#balancing-egress-traffic-to-upstream).

When a KrakenD endpoint is hit, the engine requests **all defined backends in parallel** (unless a [sequential proxy](/docs/v2.11/endpoints/sequential-proxy/) is used) and the content [merged and aggregated](/docs/v2.11/endpoints/response-manipulation/#aggregation-and-merging) (unless you just proxy using the [`no-op` encoding](/docs/v2.11/endpoints/no-op/)). The returned content is parsed according to its `encoding` or in some cases its `extra_config` configuration.

{{< note title="Returned status codes and headers" type="info" >}}
Because KrakenD is not a reverse proxy, the status code and headers returned depends on factors like the type of encoding and the configuration you have chosen. See [Status Codes](/docs/v2.11/endpoints/status-codes/) and (Returning the backend headers and errors)[/docs/backends/detailed-errors/] for more details.
{{< /note >}}



## Backend/Upstream configuration
Inside the `backend` array, you need to create an object for each upstream service used by its declaring endpoint. The combination of `host` + `url_pattern`set the full URL that KrakenD will use to fetch your upstream services. Most of the backends will require a simple configuration like:
```json
{
    "host": ["http://your-api"],
    "url_pattern": "/url"
}
```

The `url_pattern` accepts `{variables}` from the endpoint definition, and on {{< badge >}}Enterprise{{< /badge >}} you can also [inject headers](/docs/enterprise/endpoints/dynamic-routing/) with patterns like `/{input_headers.X-Tenant}/foo`


All the options relative to the **backend definition** are:

{{< schema version="v2.11" data="backend.json" >}}

Other configuration options such as the ones for [data manipulation](/docs/v2.11/backends/data-manipulation/) are available. You will find them in each specific feature section.

## Backend configuration example
In the example below, KrakenD offers an endpoint `/v1/products` that merges the content from two different services using the URLs `/products` and `/offers`. The marketing (`marketing.myapi.com`) and the products (`products-XX.myapi.com`) API requests are fired simultaneously. KrakenD will load-balance among the listed hosts (here or in your [service discovery](/docs/v2.11/backends/service-discovery/)) to pick one of the three hosts.

```json
{
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
        }
    ]
}
```

## Connecting to HTTPS backends with self-signed certificates
When using **self-signed certificates** in your backends, you must add the certificates to the local CA, or at least add them while developing the `allow_insecure_connections` setting to `true`. Example:

```json
{
  "version": 3,
  "client_tls": {
        "allow_insecure_connections": true
  },
  "endpoints": []
}
```
