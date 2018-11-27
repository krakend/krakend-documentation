---
lastmod: 2018-11-27
date: 2018-07-20
aliases:
- /docs/features/parameter-forwarding/
linktitle:  Parameter forwarding
title: Parameter forwarding
weight: 30
menu:
  documentation:
    parent: endpoints
---
KrakenD is an API Gateway, and when it comes to forward query strings, cookies, and headers, it **does not behave like a regular proxy** by forwarding parameters to the backend.

The **default policy** for data forwarding works as follows:

- No [query string](#query-string-forwarding) parameters are forwarded to the backend
- No [headers](#headers-forwarding) are forwarded
- No [cookies](#cookies-forwarding) are forwarded

You can change this behavior according to your needs, and define which elements are allowed to pass.

However, if you want a specific endpoint to act just like a regular proxy without doing any operations use the `no-op` option on its encoding options. When using `no-op`, the gateway can't probe the content whatsoever. The request goes from the client to the backend "as is".

# Query string forwarding
KrakenD **sends all query string parameters to the backend by default**. Meaning that if an endpoint `/foo` receives the query string `/foo?a=1&b=2` all its declared backends are going to receive `a` and `b`.

Nevertheless, it's recommended to add the property list `querystring_params` in the `endpoint` configuration to declare the allowed parameters and avoid polluting backends. When this list exists in the configuration, the forwarding policy behaves like a whitelist: all matching parameters declared in the `querystring_params` list are forwarded to the backend, and the rest dropped.

For instance, let's forward `?a=1&b=2` only to the backends:

    {
      "version": 2,
      "endpoints": [
        {
          "endpoint": "/v1/foo",
          "method": "GET",
          "querystring_params": [
            "a",
            "b"
          ],
          "backend": [
            {
              "url_pattern": "/catalog",
              "sd": "static",
              "host": [
                "http://some.api.com:9000"
              ]
            }
          ]
        }
      ]
    }

With this configuration, given a request like `http://krakend:8080/v1/foo?a=1&b=2&evil=here`, the backend receives `a` and `b`, but `evil` is missing.

Also, if a request like `http://krakend:8080/v1/foo?a=1` is skipping an allowed parameter like `b`, this parameter is missing in the backend as well.

## Mixing endpoint {variables} and query string parameters
The `{variables}` used in the endpoints definition can be injected in the backends as part of the query string parameters as well. These variables combine with any actual query string parameters in the endpoint. For instance:

    ...
    {
            "endpoint": "/v3/{channel}/foo",
            "backend": [
                    {
                            "host": ["http://backend"],
                            "url_pattern": "/foo?channel={channel}"
                    }
            ]
    }

{{% note title="Important note" %}}
Injecting variables in the backend as a query string enables the forwarding of all client query string parameters, unless the `querystring_params` is set.
{{% /note %}}

With the configuration above a request to the KrakenD endpoint such as `http://krakend/v3/iOS/foo?limit=10&evil=here` makes a call to the backend with the following query string parameters:

    /foo?channel=iOS&limit=10&evil=here

The same call using `querystring_params` produces the equivalent behavior, only that unexpected parameters drop:

    {
            "endpoint": "/v3/{channel}/foo",
            "querystring_params": [
                    "page",
                    "limit"
            ],
            "backend": [
                    {
                            "host": ["http://backend"],
                            "url_pattern": "/foo?channel={channel}"
                    }
            ]
    }

And the backend receives:

    /foo?channel=iOS&limit=10

No `evil` here! So, remember to set always the `querystring_params` if you intend to inject variables as parameters.

Read the [`/__debug/` endpoint](/endpoints/debug-endpoint) to understand how to test query string parameters.

# Headers forwarding
KrakenD **does not send client headers to the backend by default**.  Use `headers_to_pass`.

Declare the list of headers sent by the client that you want to let pass to the backend with the `headers_to_pass` option.

A client request from a browser or a mobile client usually contains a lot of headers, including cookies. Typical examples of the variety of headers that clients send are `Host`, `Connection`, `Cache-Control`, `Cookie`... and a long, long etcetera. The backend usually does not need any of this to return the content.

KrakenD passes only these essential headers to the backends:

    Accept-Encoding: gzip
    Host: localhost:8080
    User-Agent: KrakenD Version {{% version %}}
    X-Forwarded-For: ::1

 When you use the `headers_to_pass`, take into account that any of these headers are replaced with the ones you declare.

 An example to pass the `User-Agent` to the backend:

    {
      "version": 2,
      "endpoints": [
        {
          "endpoint": "/v1/foo",
          "method": "GET",
          "headers_to_pass": [
            "User-Agent"
            ],
          "backend": [
            {
              "url_pattern": "/catalog",
              "sd": "static",
              "host": [
                "http://some.api.com:9000"
              ]
            }
          ]
        }
      ]
    }

This setting changes the headers received by the backend to:

    Accept-Encoding: gzip
    Host: localhost:8080
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
    X-Forwarded-For: ::1

Read the [`/__debug/` endpoint](/endpoints/debug-endpoint) to understand how to test headers.

# Cookies forwarding
A cookie is just some content passing inside the `Cookie` header. If you want cookies to reach your backend, add the `Cookie` header under `headers_to_pass`, just as you would do with any other header.

When doing this, **all your cookies** are sent to all backends inside the endpoint. Use this option wisely!

Example:

    {
      "version": 2,
      "endpoints": [
        {
          "endpoint": "/v1/foo",
          "method": "GET",
          "headers_to_pass": [
            "Cookie"
            ],
          "backend": [
            {
              "url_pattern": "/catalog",
              "sd": "static",
              "host": [
                "http://some.api.com:9000"
              ]
            }
          ]
        }
      ]
    }
