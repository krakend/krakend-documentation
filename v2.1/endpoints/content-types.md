---
lastmod: 2022-09-14
old_version: true
date: 2016-04-14
linktitle: Output encoding
title: Output encoding
weight: 10
menu:
  community_v2.1:
    parent: "040 Endpoint Configuration"
images:
- /images/documentation/diagrams/content-types.mmd.svg
skip_header_image: true
---

An important concept to get familiar with is that **by default, KrakenD does not work as a reverse proxy** (unless you use the [`no-op` encoding](/docs/v2.1/endpoints/no-op/)). When clients consume upstream services content through KrakenD, it is **automatically transformed to the encoding of your choice**, and you have the opportunity to manipulate and aggregate data easily.

KrakenD can send responses back to the client **in a different format** than what your services provide. We call the encoding you provide to the end-user the `output_encoding`, while the content your services (backend) provide KrakenD we call them `encoding`.

**The response flow is**:

- The `encoding` is how KrakenD expects to find the response data of your backends. It is declared in the [`backend` section](/docs/v2.1/backends/supported-encodings/)
- The `output_encoding` is how you would like to process and return the responses to the client. It is declared in the `endpoint` section.

After KrakenD has queried your services, the flow looks like this:

![content-type-flow.mmd diagram](/images/documentation/diagrams/content-type-flow.mmd.svg)

## Example
For instance, you can have one endpoint `/foo` that consumes content from multiple services in parallel in different formats (`encoding`) like  XML or RSS. But you want to return the aggregated information in JSON (the `output_encoding`). You can mix encodings and return them normalized automatically.

![Output encoding diagram](/images/documentation/diagrams/content-types.mmd.svg)


## Configuration of `output_encoding`
The following `output_encoding` strategies are available to choose from for every endpoint, depending on the decoding and encoding needs you have:

### Proxy to one service
- `no-op`: No operation, meaning that KrakenD skips any encoding or decoding, capturing whatever content, format, and status code your backend returns. This is how most API gateway products work today, but KrakenD is not just a proxy. [See no-op documentation](/docs/v2.1/endpoints/no-op/).

### Working with JSON

- `json`: This is the **default encoding** when no `output_encoding` is declared or when you pass an invalid option. The endpoint always returns a JSON object to the client, no matter what the `encoding` of your backend is.
- `fast-json`: Same as `json` but it's ~140% faster on collections and ~30% faster on objects (average tests). Only available on the Enterprise Edition. You will notice the difference in speed of the fast-json encoding when the payloads increase in size (a small payload has an insignificant comparison to `json` encoding).
- `json-collection`: Returning an array or collection is not treated equally to an object. When the endpoint must return a JSON collection `[...]` instead of an object `{...}`, you must use this output. The backend response expects an object named `collection`, but this is automatically done by KrakenD when you use in the `backend` the [`is_collection` or `safejson`](/docs/v2.1/backends/supported-encodings/).

### Working with non-JSON

- `xml`: When the endpoint returns an XML object no matter the encoding of your backend.
- `string`: Treat the whole response as a simple string
- `negotiate`: Allows the client to choose by parsing its `Accept` header. KrakenD accepts:
  - `application/json`
  - `application/xml`
  - `text/plain` (outputs in YAML)

## Output encoding examples
Each endpoint declaration can define which encoder should be used, as shown in this example. By default, when the `output_encoding` is omitted, KrakenD falls back to JSON:

```json
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
```
The endpoint `/baz` will use the default encoding `json` as no encoding has been defined.
