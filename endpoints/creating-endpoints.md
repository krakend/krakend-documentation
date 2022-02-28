---
lastmod: 2021-01-28
date: 2020-02-26
linktitle:  Creating endpoints
title: How to create KrakenD endpoints
weight: -10
menu:
  community_current:
    parent: "040 Endpoint Configuration"
---
KrakenD endpoints are the essential part of KrakenD as they are what your end users consume.

See [Understanding the configuration file](/docs/configuration/structure/) if you haven't read it yet.

To create an endpoint you only need to add an **endpoint object** under the `endpoints` list with the resource you want to expose. If no `method` is declared, it's assumed to be read-only (`GET`).

The endpoints section looks like this:

{{< highlight json >}}
{
  "endpoints": [
    {
      "endpoint": "/v1/foo",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/bar",
          "method": "GET",
          "host": [
            "https://api.mybackend.com"
          ]
        }
      ]
    }
  ]
}
{{< /highlight >}}


The previous example exposes to the clients a `GET /v1/foo` endpoint and takes the data from your existing backend `GET https://api.mybackend.com"/bar`. As there is no other additional configuration, the data won't be manipulated.

## Attributes
The **endpoint object** accepts the following attributes:

- `endpoint`: The resource URL you want to expose
- `method` (*optional*): Must be **written in uppercase** `GET`, `POST`, `PUT`, `PATCH`, `DELETE`. Defaults to `GET`.
- `output_encoding`: See the [supported encodings](/docs/endpoints/content-types/). Defaults to `json`.
- `backend`: List of all the **backend objects** queried for this endpoint.
- `extra_config` (*optional*): Configuration of components and middlewares that are executed with this endpoint.
- `input_query_strings` (*optional*): Recognized GET parameters. See [parameter forwarding](/docs/endpoints/parameter-forwarding/).
- `input_headers` (*optional*): Forwarded headers. See [headers forwarding](/docs/endpoints/parameter-forwarding/#headers-forwarding).
- `concurrent_calls` (*optional*): A technique to improve response times. See [concurrent requests](/docs/endpoints/concurrent-requests/)
- `cache_ttl` (*optional*): (*time unit*) The cache headers informing for how long the CDN can cache the request to this endpoint. Related: [caching backend responses](/docs/backends/caching/).
- `timeout` (*optional*): (*time unit*) Maximum time you'll wait for the slowest backend response. Usually specified in seconds (`s`) or milliseconds (`ms`. E.g: `1500ms`)

\* Valid _time units_ are: `ns`, `us`, (or `µs`), `ms`, `s`, `m`, `h` E.g: `1s`


### Multiple methods of the same resource

The `method` key defines the HTTP verb you can use with the endpoint. You need to declare **one endpoint object for each method**. So if you want the same endpoint to listen to `GET` and `POST` requests you need the following configuration:
{{< highlight json "hl_lines=4 5 18 19">}}
{
  "endpoints": [
    {
      "endpoint": "/v1/foo",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/bar",
          "method": "GET",
          "host": [
            "https://api.mybackend.com"
          ]
        }
      ]
    },

    {
      "endpoint": "/v1/foo",
      "method": "POST",
      "backend": [
        {
          "url_pattern": "/bar",
          "method": "POST",
          "host": [
            "https://api.mybackend.com"
          ]
        }
      ]
    }

  ]
}
{{< /highlight >}}
Notice that the `method` is declared both in the endpoint and in the backend (as they could be different).

### Endpoint variables

Endpoints can define variables in its endpoint definition. To do so, encapsulate the variable name with curly braces, like `{var}`.

{{< highlight json >}}
{
  "endpoint": "/user/{id}"
}
{{< /highlight >}}


The previous endpoint will accept requests like `/user/123` or `/user/A-B-C` or anything else while it does not contain another slash. A request like `/user/1/2` won't be recognized by this endpoint. For such scenario you need to declare `/user/{first}/{second}`.

### Limitations
URLs do not support colons `:` in its definition.
