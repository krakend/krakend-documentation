---
lastmod: 2023-03-21
date: 2018-11-27
linktitle: Echo endpoint
menu:
  community_current:
    parent: "040 Endpoint Configuration"
title: The `/__echo/` endpoint
description: The echo endpoint is a simple debugging tool that replies with an object containing the details of a request. Use it to debug configurations and test components
weight: 35
notoc: true
---
The `/__echo/` endpoint is a developer tool to help you debug configurations. It works similarly to the [`/__debug/` endpoint](/docs/endpoints/debug-endpoint/), but instead of printing the requests in the log and returning a `{"message": "pong"}`, they are printed in the response. It replies with an object containing all the request details, and you can use it as an endpoint or backend. As KrakenD has a zero-trust approach, you will find out the exact information that passes through in this endpoint.

## Configuration
To enable the `/__echo/` endpoint, you should add in the configuration (service level) the flag `echo_endpoint`, and then use it directly by calling `http://krakend:8080/__echo/` or by adding it as a `backend` in any endpoint.

{{< schema data="v3.json" filter="echo_endpoint">}}

When used as a backend, you have a **fake backend** that is very useful for seeing the interaction between the gateway and the backends and testing all sorts of KrakenD components.

### Response fields
Given a request following the format `[scheme:][//[userinfo@]host][/]path[?query][#fragment]`, the `/__echo/` endpoint will answer with the following structure:

```json
{
  "req_uri": "http://userinfo@krakend:8080/__echo/foo/bar/vaz?q=foo#fragment",
  "req_uri_details": {
    "scheme": "http",
    "user": "userinfo",
    "host": "krakend:8080",
    "path": "/__echo/foo/bar/vaz",
    "query": "?q=foo",
    "fragment": "fragment"
  },
  "req_method": "POST",
  "req_querystring": {
     "q": ["foo"]
  },
  "req_body": {
     "@comment": "The correct parsing of the body is not guaranteed as its content is unknown (and even binary)"
  },
  "req_headers": {
     "Content-Type": ["application/json"]
  }
}
```

## Echo endpoint example
The most beneficial case is when you add KrakenD itself as another backend using the `/__echo/` endpoint. Then, you can see exactly what headers and query string parameters your backends receive in the responses.

To test it, save the content of this file in a `krakend.json` and start the server:

```json
{
  "version": 3,
  "port": 8080,
  "echo_endpoint": true,
  "endpoints": [
    {
      "endpoint": "/test/{var}",
      "backend": [
        {
          "host": ["http://127.0.0.1:8080"],
          "url_pattern": "/__echo/{var}"
        }
      ]
    }
  ]
}
```

**Default behavior:**

{{< terminal title="Ignore query strings by default">}}
curl -i -H'Test: foo' 'http://localhost:8080/test/one?a=1&b=2&c=3'
{{< /terminal >}}

In the response, you will see that `a`, `b`, and `c` do not appear, neither the sent headers. The `curl` command automatically sends the `Accept` and `User-Agent` headers, but they are not in the backend call either. Instead, you see the KrakenD User-Agent as set by the gateway.

Play now with the [parameter forwarding](/docs/endpoints/parameter-forwarding/), such as the `input_headers` and `input_query_strings`, and repeat the test.