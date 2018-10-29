---
lastmod: 2018-10-20
date: 2016-04-14
toc: true
linktitle: Supported encodings
title: Supported backend encodings
weight: 40
menu:
  documentation:
    parent: backends
---

KrakenD can parse responses from mixed backends that are using several content types or encodings, such as:

- JSON
- XML
- RSS
- Treat as string

Each backend declaration can set a different encoder to process the responses, and still, KrakenD can transparently work with the mixed content returning a unified encoding in the endpoint.

The following example demonstrates how an endpoint `/abc` is feeding on three different services and urls  `/a`, `/b`, and `/c` and aggregates their responses:

	...
	"endpoints": [
    {
      "endpoint": "/abc",
      "backend": [
        {
          "url_pattern": "/a",
          "encoding": "json",
          "host": [
            "http://service-a.company.com"
          ]
        },
        {
          "url_pattern": "/b",
          "encoding": "xml",
          "host": [
            "http://service-b.company.com"
          ]
        },
        {
          "url_pattern": "/c",
          "encoding": "rss",
          "host": [
            "http://service-c.company.com"
          ]
        }
      ]
    }
    ...

As you can see, having the `encoding` declaration inside every backend allows you to consume services with different content types. The endpoint `/abc` instead uses the encoding of your choice (e.g., JSON).

# Response content type

KrakenD supports sending responses back to the client using content types other than JSON (v0.5 or greater). The list of supported content types depends on the selected router package.

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
