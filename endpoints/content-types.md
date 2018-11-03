---
aliases:
- /features/other/
- /docs/features/content-types/
lastmod: 2018-10-20
date: 2016-04-14
toc: true
linktitle: Content Types
title: Response content types
weight: 50
menu:
  documentation:
    parent: endpoints
---

KrakenD supports sending responses back to the client using content types other than JSON (v0.5 or greater. The list of supported content types depends on the selected router package.

Each endpoint declaration is able to define which encoder should be used, as shown in this example. By default, the KrakenD router will fall back to JSON:

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

## Mux

The three mux based routers supported by the KrakenD framework include these output encodings:

- JSON
- String

## Gin

The gin-based KrakenD router includes these output encodings:

- JSON
- String
- Negotiate: internally supports JSON, XML, and YAML. It selects one or the other depending on the received `Accept` header.
