---
lastmod: 2022-03-21
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
The concept of `backend` refers to the origin servers providing the necessary data to populate your endpoints. A backend can be something like your HTTP-based API, a Lambda function, or a Kafka queue, to name a few examples.

A `backend` can be any server inside or outside your network, as long it is reachable by KrakenD. For instance, you can create endpoints fetching data from your internal servers and enrich them by adding third-party data from an external API like Github, Facebook, or other services. You can also return everything aggregated in a single glorified response.

A `backend` object is an array of all the services that an [`endpoint`](/docs/endpoints/) connects to. It defines the list of hostnames that connects to and the URL to send or receive the data.

When a KrakenD endpoint is hit, the engine requests **all defined backends in parallel** (unless a [sequential proxy](/docs/endpoints/sequential-proxy/) is used, or you use [`no-op` encoding](/docs/endpoints/no-op/)). The returned content is parsed according to its `encoding` or in some cases its`extra_config` configuration.

{{< note title="Multiple non-safe methods" type="info" >}}
Despite you can use several backends in one endpoint, KrakenD **does not allow you to define multiple non-safe (write) backends**. This is a (sometimes controversial) design decision to disable the gateway to handle transactions.

If you need to have a write method (POST, PUT, DELETE, PATCH) together with other GET methods, use the [sequential proxy](/docs/endpoints/sequential-proxy/) and place a maximum of 1 write method at the end of the sequence.
{{< /note >}}


## Backend/Upstream configuration
Inside the `backend` array, you need to create an object for each upstream service used by its declaring endpoint. The combination of `host` + `url_pattern`set the full URL that KrakenD will use to fetch your upstream services. Most of the backends will require a simple configuration like:
{{< highlight json >}}
{
    "host": ["http://your-api"],
    "url_pattern": "/url"
}
{{< /highlight >}}


The options relative to the **backend definition** are:

- `host` (*array* - required): An array with all the available hosts to **load balance** requests, including the schema (when possible) `schema://host:port`. E.g.: ` https://my.users-ms.com`. If you are in a platform where hosts or services are balanced (e.g., a K8S service), write a single name in the array with the service name/balancer address. Defaults to the `host` declaration at the configuration's root level, and KrakenD fails starting when none.
- `url_pattern` (*string* - required): The path inside the service (no protocol, no host, no method). E.g: `/users`. Some functionalities under `extra_config` might drop the requirement of declaring an `url_pattern`. The URL must be RESTful, if it is not (e.g.: `/url.{some_variable}.json`) see below how to [disable RESTful checking](#disable-restful-checking).
- `encoding` (*string* - optional): Define your [needed encoding](/docs/backends/supported-encodings/) to inform KrakenD how to parse the response. Defaults to the value of its endpoint's `encoding`, or to `json` if not defined anywhere else.
- `sd` (*string* - optional): The service [Service Discovery](/docs/backends/service-discovery/) system to resolve your backend services. Defaults to `static` (no external Service Discovery). Use `dns` to use DNS SRV records.
- `method` (*string* - optional): One of `GET`, `POST`, `PUT`, `DELETE`, `PATCH` (in **uppercase**!). The method does not need to match the endpoint's method.
- `disable_sanitize` (*boolean* - optional): Set to `true` when the host doesn't need to be checked for an HTTP protocol. This is the case of `sd=dns` or when using other protocols like `amqp://`, `nats://`, `kafka://`, etc. When set to true, and the protocol is not http, KrakenD fails with `invalid host` error. Defaults to `false`.
- `extra_config` (*object* - optional ): When there is additional configuration related to a specific component or middleware (like a circuit breaker, rate limit, etc.) it is declared under this section.

Other configuration options such as the ones for [data manipulation](/docs/backends/data-manipulation/) are available. You will find them in each specific feature section.

## Backend configuration example
In the example below, KrakenD offers an endpoint `/v1/products` that merges the content from two different services using the URLs `/products` and `/offers`. The marketing (`marketing.myapi.com`) and the products (`products-XX.myapi.com`) API requests are fired simultaneously. KrakenD will load balance among the listed hosts (here or in your [service discovery](/docs/backends/service-discovery/)) to pick one of the three hosts.

{{< highlight json >}}
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
{{< /highlight >}}




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
