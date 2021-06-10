---
lastmod: 2018-11-27
date: 2018-07-20
aliases:
- /docs/features/parameter-forwarding/
linktitle:  Parameter forwarding
title: Parameter forwarding
weight: 30
menu:
  community_v1.3:
    parent: "040 Endpoint Configuration"
---
KrakenD is an API Gateway, and when it comes to forward query strings, cookies, and headers, it **does not behave like a regular proxy** by forwarding parameters to the backend.

The **default policy** for data forwarding works as follows:

- No [query string](#query-string-forwarding) parameters are forwarded to the backend
- No [headers](#headers-forwarding) are forwarded
- No [cookies](#cookies-forwarding) are forwarded

You can change this behavior according to your needs, and define which elements are allowed to pass.

## Optional query string forwarding
KrakenD **does not send any query string parameter to the backend by default**, avoiding the pollution of the backends. Meaning that if an endpoint `/foo` receives the query string `/foo?a=1&b=2` all its declared backends are not going to see neither `a` nor `b`.

The property list `querystring_params` in the `endpoint` configuration allows you to declare **optional query string parameters**. When this list exists in the configuration, the forwarding policy behaves like an allow list: all matching parameters declared in the `querystring_params` list are forwarded to the backend, and the rest dropped.

Parameters are always optional and the user can pass a subset of them, all, or none.

For instance, let's forward `?a=1&b=2` to the backends:

    {
      "version": 2,
      "endpoints": [
        {
          "endpoint": "/v1/foo",
          "querystring_params": [
            "a",
            "b"
          ],
          "backend": [
            {
              "url_pattern": "/catalog",
              "host": [
                "http://some.api.com:9000"
              ]
            }
          ]
        }
      ]
    }

With this configuration, given a request like `http://krakend:8080/v1/foo?a=1&b=2&evil=here`, the backend receives `a` and `b`, but `evil` is missing.

Also, if a request like `http://krakend:8080/v1/foo?a=1` does not include `b`, this parameter is simply missing in the backend request as well.

### Sending all query string parameters
While the default policy prevents from sending unrecognized query string parameters, setting an asterisk `*` as the parameter name makes the gateway to **forward any query string to the backends**:
```
"querystring_params":[  
      "*"
]
```
Enabling the wildcard pollutes your backends, as any query string sent by end users or malicious attackers gets through the gateway and impacts the backends behind. Our recommendation is to let the gateway know which are the query strings in the API contract and specify them in the list, even when the list is long, and not use the wildcard. If the decision is to go with the wildcard, make sure your backends can handle abuse attempts from clients.

### Mandatory query string parameters
When your backend requires query string parameters and you want to make them **mandatory** in KrakenD, use the `{variables}` placeholders in the endpoints definition. The variables can be injected in the backends as part of the query string parameters. For instance:

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

The parameter is mandatory as if a value for `channel` is not provided the server replies with a `404`.

With the configuration above a request to the KrakenD endpoint such as `http://krakend/v3/iOS/foo?limit=10&evil=here` makes a call to the backend with only the `channel` query string:

    /foo?channel=iOS

Nevertheless, the `querystring_params` could also be added in this configuration, creating a special case of optional and mandatory parameters! You would be passing query strings both hardcoded in the `url_pattern` and generated from the user input. What happens in this strange case is that if the user passes a single optional query string parameter that is declared in `querystring_params` then the mandatory value is lost. If the request does not contain any known optional parameter, then the mandatory value is used. For instance:

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

With `http://krakend/v3/iOS/foo?limit=10&evil=here` the backend receives:

    /foo?limit=10

No mandatory `channel` here! Because the optional parameter `limit` has been declared.

On the other hand, `http://krakend/v3/iOS/foo?evil=here` produces:

	 /foo?channel=iOS

No optional parameter has been passed, so the mandatory one is used.

Read the [`/__debug/` endpoint](/docs/v1.3/endpoints/debug-endpoint/) to understand how to test query string parameters.

## Headers forwarding
KrakenD **does not send client headers to the backend by default**.  Use `headers_to_pass`.

Declare the list of headers sent by the client that you want to let pass to the backend with the `headers_to_pass` option.

A client request from a browser or a mobile client usually contains a lot of headers, including cookies. Typical examples of the variety of headers that clients send are `Host`, `Connection`, `Cache-Control`, `Cookie`... and a long, long etcetera. The backend usually does not need any of this to return the content.

KrakenD passes only these essential headers to the backends:

- `Accept-Encoding`
- `Host`
- `User-Agent` (KrakenD Version {{< version >}})
- `X-Forwarded-For`
- `X-Forwarded-Host`
- `X-Forwarded-Via` (only when `User-Agent` is in the `headers_to_pass`)

When you use the `headers_to_pass`, take into account that any of these headers are replaced with the ones you declare.

 An example to pass the `User-Agent` to the backend:

    {
      "version": 2,
      "endpoints": [
        {
          "endpoint": "/v1/foo",
          "headers_to_pass": [
            "User-Agent"
            ],
          "backend": [
            {
              "url_pattern": "/catalog",
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

Read the [`/__debug/` endpoint](/docs/v1.3/endpoints/debug-endpoint/) to understand how to test headers.

### Sending all client headers to the backends
While the default policy prevents forwarding unrecognized headers, setting an asterisk `*` as the parameter name makes the gateway to **forward any header to the backends**, including cookies:
```
"headers_to_pass":[  
      "*"
]
```
Enabling the wildcard pollutes your backends, as any header sent by end users or malicious attackers gets through the gateway and impacts the backends behind. Our recommendation is to let the gateway know which are the headers in the API contract and specify them in the list, even when the list is long, and not use the wildcard. If the decision is to go with the wildcard, make sure your backends can handle abuse attempts from clients.


## Cookies forwarding
A cookie is just some content passing inside the `Cookie` header. If you want cookies to reach your backend, add the `Cookie` header under `headers_to_pass`, just as you would do with any other header.

When doing this, **all your cookies** are sent to all backends inside the endpoint. Use this option wisely!

Example:

    {
      "version": 2,
      "endpoints": [
        {
          "endpoint": "/v1/foo",
          "headers_to_pass": [
            "Cookie"
            ],
          "backend": [
            {
              "url_pattern": "/catalog",
              "host": [
                "http://some.api.com:9000"
              ]
            }
          ]
        }
      ]
    }
