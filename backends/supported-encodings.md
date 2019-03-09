---
lastmod: 2019-03-08
date: 2016-04-14
toc: true
linktitle: Supported encodings
title: Supported backend encodings
weight: 40
notoc: true
menu:
  documentation:
    parent: backends
---

KrakenD can parse responses from mixed backends that are using several content types or encodings, such as:

- JSON
- XML
- RSS
- Treat as string

Additionally the special case [No-op (proxy)](/docs/endpoints/no-op/) is available but cannot be used to merge content.


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
