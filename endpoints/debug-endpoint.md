---
lastmod: 2018-11-27
date: 2018-11-27
linktitle: Debug endpoint
menu:
  documentation:
    parent: endpoints
title: The `/__debug` endpoint
weight: 35
---
The `/__debug` endpoint is available when you start the server with the `-d` flag.

The endpoint can be used as a fake backend to see its activity in the log. When developing, add KrakenD itself as a backend using the `/__debug/` endpoint so you can see exactly what headers and query string parameters your backends are receiving.

The debug endpoint might save you a lot of trouble, as your application might not work when certain headers or parameters are not present, and you might be relying in what your client is sending, and not in what the gateway is sending.

For instance, your client might be sending a `Content-Type` header, but unless this header is recognized by the gateway (added in `headers_to_pass`), it is not going to reach the backend. Seeing the exact headers and parameters in the log clear all the doubts and you can reproduce the call and conditions easily.

# Debug endpoint configuration example
The following configuration demonstrates how to test what headers and query string parameters are sent and received by the backends.

We are going to test the following endpoints:

- `/default-behavior`: No client headers, query string or cookies are forwarded.
- `/known-params`: Forwards known parameters and headers
    - Recognizes `a` and `b` as query string
    - Recognizes `User-Agent` as forwarded header

To test it right now, save the content of this file in a `krakend-test.json` and start the server with the `-d` flag:

    {
        "version": 2,
        "port": 8080,
        "endpoints": [
            {
            "endpoint": "/default-behavior",
            "method": "GET",
            "backend": [
                {
                "url_pattern": "/__debug/all",
                "host": [ "http://127.0.0.1:8080" ]
                }
            ]
            },
            {
            "endpoint": "/known-params",
            "method": "GET",
            "querystring_params": [
                "a",
                "b"
            ],
            "headers_to_pass": [
                "User-Agent",
                "Accept"
            ],
            "backend": [
                {
                "url_pattern": "/__debug/some",
                "host": [ "http://127.0.0.1:8080" ]
                }
            ]
            }
        ]
    }

Start the server:

    krakend run -d -c krakend-test.json


Now we can test that the endpoints behave as expected

**Default behavior:**

    curl -i 'http://localhost:8080/default-behavior?a=1&b=2&c=3'

The `curl` command automatically sends the `Accept` and `User-Agent` headers. And then in the KrakenD log we can see that `a`, `b`, and `c` are not forwarded, neither its headers.

{{< highlight go "hl_lines=5 8" >}}
DEBUG: Method: GET
DEBUG: URL: /__debug/all
DEBUG: Query: map[]
DEBUG: Params: [{param /all}]
DEBUG: Headers: map[User-Agent:[KrakenD Version 0.7.0] X-Forwarded-For:[::1] Accept-Encoding:[gzip]]
DEBUG: Body:
[GIN] 2018/11/27 - 22:32:44 | 200 |     118.543µs |             ::1 | GET      /__debug/all
[GIN] 2018/11/27 - 22:32:44 | 200 |     565.971µs |             ::1 | GET      /default-behavior?a=1&b=2&c=3
{{< /highlight >}}

Now let's repeat the same with the known parameters:

    curl -i 'http://localhost:8080/known-params?a=1&b=2&c=3'

And now in the KrakenD log now we can see that the `User-Agent` and `Accept` are present:

{{< highlight go "hl_lines=5 8" >}}
DEBUG: Method: GET
 DEBUG: URL: /__debug/some?a=1&b=2
 DEBUG: Query: map[a:[1] b:[2]]
 DEBUG: Params: [{param /some}]
 DEBUG: Headers: map[User-Agent:[curl/7.54.0] Accept:[*/*] X-Forwarded-For:[::1] Accept-Encoding:[gzip]]
 DEBUG: Body:
[GIN] 2018/11/27 - 22:33:23 | 200 |     122.507µs |             ::1 | GET      /__debug/some?a=1&b=2
[GIN] 2018/11/27 - 22:33:23 | 200 |     542.483µs |             ::1 | GET      /known-params?a=1&b=2&c=3
{{< /highlight >}}



