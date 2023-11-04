---
lastmod: 2018-11-28
date: 2018-11-27
linktitle: Debug endpoint
menu:
  community_current:
    parent: "040 Endpoint Configuration"
title: Debug Endpoint in KrakenD API Gateway
description: Learn how to utilize the debug endpoint in KrakenD API Gateway for troubleshooting and debugging purposes during API development and testing
weight: 35
notoc: true
---
The `/__debug/` endpoint is available when you start the server with the `-d` flag, or when you add the `debug_endpoint=true` in the configuration.

The endpoint can be used as a **fake backend** if you use as `host` KrakenD itself, and is very useful to see the interaction between the gateway and the backends as its activity is printed in the log using the `DEBUG` log level. The endpoint returns this content:

```json
{
  "message": "pong"
}
```


When developing, add an additional `backend` pointing to KrakenD itself (`"host": "http://localhost:8080"`) with the `/__debug/` endpoint in its `url_pattern`, so you can see exactly what headers and query string parameters your backends are receiving.

The debug endpoint might save you much trouble, as your application might not work when specific headers or parameters are not present. Maybe you are relying upon what your client is sending, but this is not what the gateway is forwarding. Remember: this is not a proxy.

For instance, your client might be sending a `Content-Type` or `Accept` header and these are perhaps necessary for the proper functioning of your backend, but unless these are recognized headers by the gateway (they are in `input_headers`), they are not going to reach the backend ever. Seeing the specific headers and parameters in the log clears all the doubts, and you can reproduce the call and conditions easily.

## Configuration
To enable the debug endpoint add the following in the configuration:

{{< schema data="krakend.json" filter="debug_endpoint">}}

Or to do it during runtime, add `-d` when starting the server. E.g., `krakend run -dc krakend.json`

## Debug endpoint example
The following configuration demonstrates how to test what headers and query string parameters are sent and received by the backends by using the `/__debug/` endpoint.

We are going to test the following endpoints:

- `/default-behavior`: No client headers, query string or cookies forwarded.
- `/optional-params`: Forwards known parameters and headers
    - Recognizes `a` and `b` as a query string
    - Recognizes `User-Agent` and `Accept` as forwarded headers
- `/mandatory/{variable}`: The query string parameters taken from a variable in the endpoint or other query string parameters

To test it right now, save the content of this file in a `krakend-test.json` and start the server:

```json
{
  "version": 3,
  "port": 8080,
  "host": ["http://127.0.0.1:8080"],
  "debug_endpoint": true,
  "endpoints": [
    {
      "endpoint": "/default-behavior",
      "backend": [
        {
          "@comment": "IMPORTANT: Notice that the /__debug uses the KrakenD host itself",
          "host": ["http://127.0.0.1:8080"],
          "url_pattern": "/__debug/default"
        }
      ]
    },
    {
      "endpoint": "/optional-params",
      "input_query_strings": [
          "a",
          "b"
        ],
      "input_headers": [
          "User-Agent",
          "Accept"
        ],
      "backend": [
        {
          "url_pattern": "/__debug/optional"
        }
      ]
    },
    {
      "endpoint": "/mandatory/{variable}",
      "backend": [
        {
          "url_pattern": "/__debug/qs?mandatory={variable}"
        }
      ]
    }
  ]
}
```


Start the server:

{{< terminal title="Run KrakenD with debug mode">}}
krakend run -d -c krakend-test.json
{{< /terminal >}}

Now we can test that the endpoints behave as expected:

**Default behavior:**

{{< terminal title="Ignore query strings by default">}}
curl -i 'http://localhost:8080/default-behavior?a=1&b=2&c=3'
{{< /terminal >}}

In the KrakenD log, we can see that `a`, `b`, and `c` do not appear in the backend call, neither its headers. The `curl` command automatically sends the `Accept` and `User-Agent` headers but they are not in the backend call either, instead we see the KrakenD User-Agent as set by the gateway:

    DEBUG: Method: GET
    DEBUG: URL: /__debug/default
    DEBUG: Query: map[]
    DEBUG: Params: [{param /default}]
    DEBUG: Headers: map[User-Agent:[KrakenD Version {{< product latest_version >}}] X-Forwarded-For:[::1] Accept-Encoding:[gzip]]
    DEBUG: Body:
    [GIN] 2018/11/27 - 22:32:44 | 200 |     118.543µs |             ::1 | GET      /__debug/default
    [GIN] 2018/11/27 - 22:32:44 | 200 |     565.971µs |             ::1 | GET      /default-behavior?a=1&b=2&c=3

Now let's repeat the same request but to the `/optional-params` endpoint:

{{< terminal title="Recognized and forwarded query strings">}}
curl -i 'http://localhost:8080/optional-params?a=1&b=2&c=3'
{{< /terminal >}}

In the KrakenD log we can see now that the `User-Agent` and `Accept` are present (as they are implicitly sent by curl), and that `a` and `b` are reaching the backend (but not `c`):

    DEBUG: Method: GET
    DEBUG: URL: /__debug/optional?a=1&b=2
    DEBUG: Query: map[a:[1] b:[2]]
    DEBUG: Params: [{param /optional}]
    DEBUG: Headers: map[User-Agent:[curl/7.54.0] Accept:[*/*] X-Forwarded-For:[::1] Accept-Encoding:[gzip]]
    DEBUG: Body:
    [GIN] 2018/11/27 - 22:33:23 | 200 |     122.507µs |             ::1 | GET      /__debug/optional?a=1&b=2
    [GIN] 2018/11/27 - 22:33:23 | 200 |     542.483µs |             ::1 | GET      /optional-params?a=1&b=2&c=3

Finally, let's note what happens when you inject mandatory query strings in the backend definition, the `/mandatory/{variable}` endpoint:

{{< terminal title="Mandatory query strings">}}
curl -i 'http://localhost:8080/mandatory/foo?a=1&b=2&c=3'
{{< /terminal >}}

As we can see, the backend includes the `?mandatory=foo` variable that was written manually in the backend definition:

    DEBUG: Method: GET
    DEBUG: URL: /__debug/qs?mandatory=foo
    DEBUG: Query: map[mandatory:[foo]]
    DEBUG: Params: [{param /qs}]
    DEBUG: Headers: map[X-Forwarded-For:[::1] Accept-Encoding:[gzip] User-Agent:[KrakenD Version 0.7.0]]
    DEBUG: Body:
    [GIN] 2018/11/28 - 19:44:19 | 200 |     210.434µs |             ::1 | GET      /__debug/qs?mandatory=foo
    [GIN] 2018/11/28 - 19:44:19 | 200 |    1.975103ms |             ::1 | GET      /mandatory/foo?a=1&b=2&c=3
