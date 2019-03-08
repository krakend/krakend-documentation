---
aliases:
- /features/other/
- /docs/features/content-types/
lastmod: 2019-03-08
date: 2016-04-14
toc: true
linktitle: Content Types
title: Response content types
weight: 50
menu:
  documentation:
    parent: endpoints
---

KrakenD supports sending responses back to the client using content types other than JSON. The list of supported content types depends on the router package used.

# Supported encodings
The gateway can work with several content types, even allowing your clients to choose how to consume the content. The following `output_encoding` strategies are available to choose for every an endpoint:

- `json`: The endpoint always return a response in JSON format to the client.
- `negotiate`: Allows the client to choose by parsing its `Accept` header. KrakenD can return:
  - JSON
  - XML
  - RSS
  - YAML.
- `string`: Treat the whole response as a simple string
- `no-op`: Proxy to ONE backend. [See its documentation](/docs/endpoints/no-op/).


Each endpoint declaration is able to define which encoder should be used, as shown in this example. By default, when the `output_encoding` is omitted, KrakenD falls back to JSON:

	...
	"endpoints": [
    {
      "endpoint": "/a",
      "output_encoding": "negotiate",
      "backend": [
        {
          "url_pattern": "/a"
        }
      ]
    },
    {
      "endpoint": "/b",
      "output_encoding": "string",
      "backend": [
        {
          "url_pattern": "/b"
        }
      ]
    },
    {
      "endpoint": "/c",
      "backend": [
        {
          "url_pattern": "/c"
        }
      ]
    }
    ...

Notice that endpoint `/c` uses JSON as no encoding has been defined.

# Using other routers from the framework
If instead of the KrakenD API Gateway (KrakenD-CE), which uses internally the gin router, you decide to build your own gateway using the KrakenD framework, the following routers and output encodings are available:

## Gin
The gin-based KrakenD router includes these output encodings:

- `json`
- `string`
- `negotiate`
- `no-op`

## Mux-based
The mux based routers supported by the KrakenD framework are:

- Mux
- Gorilla
- Negroni
- Chi
- httptreemux

and they include these output encodings:

- `json`
- `string`
- `no-op`


