---
lastmod: 2020-07-12
old_version: true
date: 2016-04-14
toc: true
linktitle: Supported encodings
title: Supported backend encodings
weight: 40
notoc: true
menu:
  community_v1.4:
    parent: "050 Backends Configuration"
---
Setting the `encoding` is an important part of the backend definition, as it informs KrakenD how to parse the responses of your services.

Each backend can reply with a different encoding and KrakenD does not have any problem working with mixed encodings at the same time. You can use the following `encoding` in each `backend` section:

- `json`
- `safejson`
- `xml`
- `rss`
- `string`
- `no-op`


Notice that all values are in **lower case**. Unknown values for `encoding` or no value at all, is treated as `json`.

Each backend declaration can set a different encoder to process the responses, and still, KrakenD can transparently work with the mixed content returning a unified encoding in the endpoint.

## How to choose the backend encoding?
Follow this table to determine how to treat your backend content:

| The backend returns...                 | Then use encoding...                |
|----------------------------------------|-------------------------------------|
| JSON inside an object (`{}`)           | `json`                              |
| JSON inside an array/collection (`[]`) | `json` with `"is_collection": true` |
| JSON with variable types               | `safejson`                          |
| XML                                    | `xml`                               |
| RSS                                    | `rss`                               |
| Not an object, but a string            | `string`                            |
| Nevermind, just proxy                  | `no-op` ([read how](/docs/v1.4/endpoints/no-op/)) |

{{< note title="Working with JSON arrays" >}}
If you want to return to the client a JSON array instead of an object, consider using the following combinations: `output_encoding: json-collection` in your `endpoint`, and `is_collection: true` in your `backend`. See [response content types](/docs/v1.4/endpoints/content-types/).
{{< /note >}}


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
