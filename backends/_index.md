---
lastmod: 2022-10-21
date: 2016-09-30
toc: true
aliases: ["/docs/backends/overview/"]
linktitle: The backend object
title: Declaring and connecting to backends
weight: -1000
notoc: true
menu:
  community_current:
    parent: "050 Backends Configuration"
---
The concept of `backend` refers to the origin servers providing the necessary data to populate your endpoints. A backend can be something like your HTTP-based API, a Lambda function, or a Kafka queue, for example.

A `backend` can be any server inside or outside your network if it is reachable by KrakenD. For instance, you can create endpoints fetching data from your internal servers and enrich them by adding third-party data from an external API like Github, Facebook, or other services. You can also return everything aggregated in a single glorified response.

A `backend` object is an array of all the services that an [`endpoint`](/docs/endpoints/) connects to. It defines the list of hostnames connected to and the URL to send or receive the data.

When a KrakenD endpoint is hit, the engine requests **all defined backends in parallel** (unless a [sequential proxy](/docs/endpoints/sequential-proxy/) is used, or you use [`no-op` encoding](/docs/endpoints/no-op/)). The returned content is parsed according to its `encoding` or in some cases its`extra_config` configuration.

{{< note title="Multiple non-safe methods" type="info" >}}
Even though you can use several backends in one endpoint, KrakenD **does not allow you to define multiple non-safe (write) backends**. This is a (sometimes controversial) design decision to disable the gateway to handle transactions.

If you need to have a write method (POST, PUT, DELETE, PATCH) together with other GET methods, use the [sequential proxy](/docs/endpoints/sequential-proxy/) and place a maximum of 1 write method at the end of the sequence.
{{< /note >}}


## Backend/Upstream configuration
Inside the `backend` array, you need to create an object for each upstream service used by its declaring endpoint. The combination of `host` + `url_pattern`set the full URL that KrakenD will use to fetch your upstream services. Most of the backends will require a simple configuration like:
```json
{
    "host": ["http://your-api"],
    "url_pattern": "/url"
}
```


All the options relative to the **backend definition** are:

{{< schema data="backend.json" >}}

Other configuration options such as the ones for [data manipulation](/docs/backends/data-manipulation/) are available. You will find them in each specific feature section.

## Backend configuration example
In the example below, KrakenD offers an endpoint `/v1/products` that merges the content from two different services using the URLs `/products` and `/offers`. The marketing (`marketing.myapi.com`) and the products (`products-XX.myapi.com`) API requests are fired simultaneously. KrakenD will load-balance among the listed hosts (here or in your [service discovery](/docs/backends/service-discovery/)) to pick one of the three hosts.

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




### Disable RESTful checking
By default KrakenD only works with **RESTful URL patterns** to connect to backends. Enable the option `disable_rest` in the root of your configuration if you have backends that aren't RESTful, e.g.: `/url.{some_variable}.json`

{{< highlight json "hl_lines=4 13">}}
{
  "$schema": "https://www.krakend.io/schema/v3.json",
  "version": 3,
  "disable_rest": true,
  "endpoints": [
    {
      "endpoint": "/foo",
      "backend": [
        {
          "host": [
            "http://mybackend"
          ],
          "url_pattern": "/url.{some_variable}.json"
        }
      ]
    }
  ]
}
{{< /highlight >}}
