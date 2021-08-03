---
aliases: ["/features/other/", "/docs/features/content-types/"]
lastmod: 2019-03-08
date: 2016-04-14
toc: true
linktitle: Content Types
title: Response content types
weight: 50
menu:
  community_current:
    parent: "040 Endpoint Configuration"
---

KrakenD supports sending responses back to the client using content types other than JSON. The list of supported content types depends on the router package used.

## Supported encodings
The gateway can work with several content types, even allowing your clients to choose how to consume the content. The following `output_encoding` strategies are available to choose for every an endpoint:

- `json`: The endpoint always returns a JSON object to the client. This is the **default encoding** if none is declared.
- `json-collection`: The endpoint returns a JSON collection (array) to the client. The backend response must  have an object named `collection`. This is always true when you use in the `backend` section the `is_collection` attribute set to `true`.
- `negotiate`: Allows the client to choose by parsing its `Accept` header. KrakenD can return:
  - JSON
  - XML
  - RSS
  - YAML.
- `string`: Treat the whole response as a simple string
- `no-op`: No operation (No encoding, and no decoding), for proxy only. [See its documentation](/docs/endpoints/no-op/).


Each endpoint declaration is able to define which encoder should be used, as shown in this example. By default, when the `output_encoding` is omitted, KrakenD falls back to JSON:

{{< highlight json >}}
{
	"endpoints": [
    {
      "endpoint": "/foo",
      "output_encoding": "negotiate",
      "backend": [
        {
          "url_pattern": "/foo"
        }
      ]
    },
    {
      "endpoint": "/var",
      "output_encoding": "string",
      "backend": [
        {
          "url_pattern": "/var"
        }
      ]
    },
    {
      "endpoint": "/baz",
      "backend": [
        {
          "url_pattern": "/baz"
        }
      ]
    }
  ]
}
{{< /highlight >}}
The endpoint `/baz` will use the default encoding `json` as no encoding has been defined.

## Using other routers (Lura Project)
If instead of the KrakenD API Gateway, which uses internally the gin router, you decide to build your own custom gateway using the [Lura Project](https://luraproject.org), the following routers and output encodings are available:

### Gin
The gin-based KrakenD router includes these output encodings:

- `json`
- `string`
- `negotiate`
- `no-op`

### Mux-based
The mux based routers supported by Lura are:

- Mux
- Gorilla
- Negroni
- Chi
- httptreemux

and they include these output encodings:

- `json`
- `string`
- `no-op`

See the list of [Lura supported routers](https://github.com/luraproject/lura/tree/master/router).
