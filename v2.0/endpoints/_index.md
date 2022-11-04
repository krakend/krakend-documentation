---
lastmod: 2021-01-28
old_version: true
date: 2020-02-26
description: Endpoints are the API URLs you expose through KrakenD. Learn how to create KrakenD endpoints and build your API programmatically.
linktitle:  The endpoint object
title: Creating API endpoints
weight: -10
menu:
  community_v2.0:
    parent: "040 Endpoint Configuration"
---
KrakenD `endpoints` are the most critical configuration part of KrakenD as they are what your end users consume. By adding endpoint objects, you create the API contract your users will consume.

{{< note title="Configuration overview" type="tip" >}}
If you are still unfamiliar with KrakenD's configuration structure, take a moment to read [Understanding the configuration file](/docs/v2.0/configuration/structure/).
{{< /note >}}

The `endpoints` array contains the **API definition you are publishing**. It is a collection of **endpoint objects**, and you have to place it at the root of your configuration file.


## The endpoint object

To create an endpoint, you only need to add an **endpoint object** under the `endpoints` collection. An endpoint object should contain at least the `endpoint` name and a `backend` section (to where it connects to). The defaults are taken if no further information is declared (e.g.: `method` will be a `GET`, and `output_encoding` as `json`).

An `endpoints` section might look like this:

{{< highlight json >}}
{
  "endpoints": [
    {
      "endpoint": "/v1/users/{user}",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/users/summary/{user}",
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


The previous example exposes to the clients a `GET /v1/users/{user}` endpoint and takes the data from your existing backend `GET https://api.mybackend.com/users/summary/{user}`.

Inside this object, you can add manipulation options and transform the response before its returned to the end-user.

### Endpoint configuration
The **endpoint object** accepts the following attributes. As you can see, most of them are optional:

- `endpoint`: The exact string resource URL you want to expose. You can use `{placeholders}` to use variables when needed. URLs do not support colons `:` in their definition.
- `backend`: List of all the [backend objects](/docs/v2.0/backends/) queried for this endpoint.
- `method` (*optional*): Must be **written in uppercase** `GET`, `POST`, `PUT`, `PATCH`, `DELETE`. Defaults to `GET`.
- `output_encoding` (*optional*): See the [supported encodings](/docs/v2.0/endpoints/content-types/). Defaults to `json`.
- `extra_config` (*optional*): Configuration of components and middlewares that are executed with this endpoint.
- `input_query_strings` (*optional*): Recognized GET parameters. See [parameter forwarding](/docs/v2.0/endpoints/parameter-forwarding/).
- `input_headers` (*optional*): Forwarded headers. See [headers forwarding](/docs/v2.0/endpoints/parameter-forwarding/#headers-forwarding).
- `concurrent_calls` (*optional*): A technique to improve response times. See [concurrent requests](/docs/v2.0/endpoints/concurrent-requests/)
- `cache_ttl` (*optional*): (*time unit*) The cache headers informing for how long the CDN can cache the request to this endpoint. Related: [caching backend responses](/docs/v2.0/backends/caching/).
- `timeout` (*optional*): (*time unit*) Maximum time you'll wait for the slowest backend response. Usually specified in seconds (`s`) or milliseconds (`ms`. E.g: `2000ms` or `2s`)

\* Valid _time units_ are: `ns`, `us`, (or `µs`), `ms`, `s`, `m`, `h` E.g: `1s`


{{< note title="An endpoint is not a prefix!" type="info" >}}
If you want the internal router to match a request URL with an endpoint, the structure must match its definition. For instance, if you want to support paths like `/user/{id}` and `/user/{id}/profile`, you need two different endpoints (explanation below)
{{< /note >}}

### Endpoints with multiple nesting levels
You might have envisioned KrakenD as a proxy and expected its `endpoint` declaration **works as a prefix** and listens to any path with an undetermined number of nesting levels. **But KrakenD does not work like this by default**. Instead, it expects you to declare every possible URL structure.

For instance, you declared and `"endpoint": "/user/{id}"` and you expected to resolve URLs like `/user/john/profile/preferences`, but you are getting a *404* instead. There are two solutions to this problem:

1. You declare all possible endpoints: `/user/{id}`, `/user/{id}/{level2}`, `/user/{id}/{level2}/{level3}`, etc.
2. You use a [Wildcard](/docs/enterprise/v2.0/endpoints/wildcard/) (Enterprise only)


### Endpoints listening to multiple methods

The `method` attribute defines the HTTP verb you can use with the endpoint. If you need to support multiple methods (e.g.,  `GET`, `POST`, `DELETE`) in the same endpoint, you will need to declare **one endpoint object for each method**. So if you want the same endpoint to listen to `GET` and `POST` requests, you need the following configuration:
{{< highlight json "hl_lines=4 5 9 17 18 22">}}
{
  "endpoints": [
    {
      "endpoint": "/v1/users/{user}",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/users/summary/{user}",
          "method": "GET",
          "host": [
            "https://api.mybackend.com"
          ]
        }
      ]
    },
    {
      "endpoint": "/v1/users/{user}",
      "method": "POST",
      "backend": [
        {
          "url_pattern": "/users/summary/{user}",
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

As you can see in the examples above, endpoints can define variables in its endpoint definition. To do so, encapsulate the variable name with curly braces, like `{var}`.

{{< highlight json >}}
{
  "endpoint": "/user/{id}"
}
{{< /highlight >}}


The previous endpoint will accept requests like `/user/123` or `/user/A-B-C`. But **it won't accept** a request like `/user/1/2`, as there is an extra slash than the definition, and KrakenD considers this to be a different endpoint.