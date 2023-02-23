---
lastmod: 2020-07-12
date: 2016-04-14
toc: true
linktitle: Supported encodings
title: Supported backend encodings
weight: 40
notoc: true
menu:
  community_current:
    parent: "050 Backends Configuration"
---
Setting the `encoding` is an important part of the backend definition, as it informs KrakenD how to parse the responses of your services.

Each backend can reply with a different encoding and KrakenD does not have any problem working with mixed encodings at the same time. You can use the following `encoding` in each `backend` section:

- `json`
- `fast-json` (Enterprise only)
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
| JSON inside an object (`{}`)           | `json` or `fast-json` (EE only)     |
| JSON inside an array/collection (`[]`) | `json` or `fast-json` with `"is_collection": true` |
| JSON with variable types               | `safejson`                          |
| XML                                    | `xml`                               |
| RSS Feed (types Atom, RSS or JSON )    | `rss`                               |
| Not an object, but a string            | `string`                            |
| Nevermind, just proxy                  | `no-op` ([read how](/docs/endpoints/no-op/)) |

{{< note title="Working with JSON arrays" >}}
If you want to return to the client a JSON array instead of an object, consider using the following combinations: `output_encoding: json-collection` in your `endpoint`, and `is_collection: true` in your `backend`. See [response content types](/docs/endpoints/content-types/).

When hesitating whether to use `safejson` or `json` and the `is_collection=true`, the `json` encoder is faster and more performant but less resilient: it will fail when the content doesn't have the expected type. If you are in control of the output of the service, choose `json`, if you are not, choose `safejson`.
{{< /note >}}

## Automatic content wrappers (`collection` and `content`)
When the content returned by your service is wrapped inside an array instead of an object, the gateway will wrap the response inside a `collection` object. This scenario is possible with `is_collection=true` , or when the `safejson` found an array in the response.

When the content returned by your service is a string, a float, integer, etc. but a type that is not an object or a collection, then the response is wrapped inside a `content` object. This scenario is possible with when the `safejson` finds a non-array or non-object type, or when you use a `string` encoding.

For instance, if your backend returns a simple `Hello World!`, the response of KrakenD (`output_encoding=json`) would be `{ "content": "Hello World!" }`.

Similarly, if your backend returns `[{"item": 1},{"item": 2}]` then the response of KrakenD (`output_encoding=json`) would be `{ "collection": [{"item": 1},{"item": 2}] }`.

Some `ouput_encodings` will revert this wrapping before returning the content to the user to match the exact value provided by the backend, but this gives you the oportunity to manipulate the content and work with it at the endpoint level. The `output_encoding` of `string` will remove the `content` wrapping, while the `json-collection` while remove the `collection` wrapping.

# Example of mixed encodings

The following example demonstrates how an endpoint `/abc` is feeding on three different services and urls  `/a`, `/b`, and `/c` and aggregates their responses. All the information is returned in JSON (`output_encoding`) despite it is consuming heterogeneous formats:

```json
{
	"endpoints": [
    {
      "endpoint": "/abc",
      "output_encoding": "json",
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
  ]
}
```

As you can see, having the `encoding` declaration inside every backend allows you to consume services with different content types. The endpoint `/abc` instead uses the encoding of your choice (e.g., JSON), but is feeding and merging from XML, RSS and JSON content simultaneously.
