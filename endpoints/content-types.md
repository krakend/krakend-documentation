---
aliases: ["/features/other/", "/docs/features/content-types/"]
lastmod: 2023-10-16
date: 2016-04-14
linktitle: Returned response formats
title: Returned encodings in KrakenD API Gateway
description: Discover how KrakenD API Gateway handles various content types, ensuring proper parsing and transformation for seamless data exchange
weight: 410
menu:
  community_current:
    parent: "060 Request and Response Manipulation"
images:
- /images/documentation/diagrams/content-types.mmd.svg
skip_header_image: true
---

An important concept to get familiar with is that by default, KrakenD **does not work as a reverse proxy** (unless you use the [`no-op` encoding](/docs/endpoints/no-op/)).

When clients consume upstream services content through KrakenD, the response is **automatically transformed to the encoding of your choice**, independently of the encoding it had in origin, and you have the opportunity to [manipulate and aggregate data](/docs/endpoints/response-manipulation/) easily.

KrakenD can send these responses back to the client **in different formats** than provided by your services (in KrakenD jargon, `backend`). We call the encoding you return to the end-user the `output_encoding`, and `encoding` the one your services return to KrakenD.

The request/response flow is:

![content-type-flow.mmd diagram](/images/documentation/diagrams/content-type-flow.mmd.svg)


- The `encoding` is how KrakenD expects to find the response data of your backends. It is declared in each [`backend` section](/docs/backends/supported-encodings/) (and you can mix types)
- The `output_encoding` is how you would like to process and return all the responses to the client. It is declared in the `endpoint` section, or globally as a default for all endpoints when you add in the root level.

**Example**: You can have an endpoint `/foo` that fetches content from multiple services in parallel in different formats (JSON, XML, RSS, etc.), and you define for each service the corresponding `encoding`. But you want to return the aggregated information in JSON (the `output_encoding`). You can mix encodings and return them normalized automatically.

![Output encoding diagram](/images/documentation/diagrams/content-types.mmd.svg)

The diagram above illustrates a gateway returning JSON content after merging multiple sources in heterogeneous formats.

## Configuration of `output_encoding`
The following `output_encoding` strategies are available to choose from for every endpoint, depending on the decoding and encoding needs you have:

### Proxy to one service
- `no-op`: No operation in the response, meaning that KrakenD skips any encoding or decoding, capturing whatever content, format, and status code your backend returns. This is how most API gateway products work today, but KrakenD is not just a proxy. [See no-op documentation](/docs/endpoints/no-op/).

### Working with JSON

- `json`: This is the **default encoding** when no `output_encoding` is declared or when you pass an invalid option. The endpoint always returns a JSON object to the client, no matter what the `encoding` of your backend is.
- `fast-json`: Same as `json` but it's ~140% faster on collections and ~30% on objects (average tests). Only available on the Enterprise Edition. You will notice the difference in speed of the fast-json encoding when the payloads increase in size (a small payload has an insignificant comparison to `json` encoding).
- `json-collection`: Returning an array or collection is not treated equally to an object. You must use this output when the endpoint must return a JSON collection `[...]` instead of an object `{...}`. The backend response expects an object named `collection`, but this is automatically done by KrakenD when you use in the `backend` the [`is_collection` or `safejson`](/docs/backends/supported-encodings/).

### Working with non-JSON

- `xml`: When the endpoint returns an XML object no matter the encoding of your backend.
- `string`: Treat the whole response as a simple string
- `negotiate`: Allows the client to choose by parsing its `Accept` header. KrakenD accepts:
  - `application/json`
  - `application/xml`
  - `text/plain` (outputs in YAML)

## Output encoding examples
Each endpoint declaration can define which encoder should be used, as shown in this example. By default, when the `output_encoding` is omitted, KrakenD falls back to the `output_encoding` in the root, or to JSON when none is declared.

```json
{
  "version": 3,
  "output_encoding": "json",
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
```
The endpoint `/baz` will use the default encoding `json` as no encoding has been defined in its definition.
