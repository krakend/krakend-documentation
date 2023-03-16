---
lastmod: 2022-06-29
old_version: true
date: 2018-07-20
linktitle:  Parameter forwarding
title: Forwarding query strings and headers
weight: 10
menu:
  community_v2.2:
    parent: "040 Endpoint Configuration"
---
KrakenD is an API Gateway with a **zero-trust policy**, and when it comes to forward query strings, cookies, and headers, you need to define what is allowed.

Part of the zero-trust policy implies that KrakenD **does not forward** any unexpected [query string](#query-string-forwarding), [headers](#headers-forwarding), or [cookies](#cookies-forwarding). See below how to set the forwarding rules.

![alt text](/images/documentation/diagrams/parameter-forwarding-1.mmd.png)

## Configuration to enable parameter forwarding
You can change the default behavior according to your needs and define which elements can pass from the client to your backends. To do that, add the following configuration options under your `endpoint` definition:

- `input_query_strings` (*array*): Defines the exact list of query strings that are allowed to reach the backend when passed
- `input_headers` (*array*): Defines the list of all headers allowed to reach the backend when passed
- A single *star* element (`["*"]`) as the value of the options above, forwards **everything** to the backend (it's safer avoiding this option)

{{< note title="Case sensitive parameters" type="info" >}}
The `input_query_strings` and `input_headers` lists are **case sensitive**. For instance, a request `?Page=1` wont pass to the backend when `"input_query_strings": ["page"]`
{{< /note >}}

**Example**:

*Send the query strings `items` and `page` to the backend, and also `User-Agent` and `Accept` headers:*

{{< highlight json "hl_lines=6-13">}}
{
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/v1/foo",
      "input_query_strings": [
        "items",
        "page"
      ],
      "input_headers": [
        "User-Agent",
        "Accept"
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
{{< /highlight >}}

Read below for further details and examples.

## Query string forwarding
The zero-trust policy implies that, for instance, if a KrakenD endpoint `/foo` receives the request `/foo?items=10&page=2`, all its declared backends are not going to see either `items` or `page`, **unless otherwise configured**.

To enable the transition of query strings to your backend, add the **list** `input_query_strings` in your `endpoint` definition. For instance, let's forward `?items=10&page=2` to the backends now:

```json
{
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/v1/foo",
      "input_query_strings": [
        "items",
        "page"
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
```

The `input_query_strings` list has the following behavior:

- **Items in the list** are forwarded to your backend when passed
- **Additional query strings not in the list** are removed from the final call
- **Writing a single *star* element** (`"input_query_strings":["*"]`) instead of individual strings, forwards **everything** to the backend

With this configuration, given a request like `http://krakend:8080/v1/foo?items=10&page=2&evil=here`, the backend receives `items` and `page`, but `evil` is missing.

Also, if a request like `http://krakend:8080/v1/foo?items=10` does not include `page`, this parameter is simply missing in the backend request as well.

By definition, query string parameters are always optional, and the user can pass a subset of them, all or none. Suppose you want to enforce that the user provides a query string parameter. In that case, you must validate it with the [Common Expression Language](/docs/v2.2/endpoints/common-expression-language-cel/) (faster) or with a [Lua script](/docs/v2.2/endpoints/lua/) (slower).

### Sending all query string parameters
While the default policy prevents from sending unrecognized query string parameters, setting an asterisk `*` as the parameter name makes the gateway to **forward any query string to the backends**:

```json
{
  "endpoint": "/foo",
  "input_query_strings":[
      "*"
  ]
}
```

**Enabling the wildcard pollutes your backends**, as any query string sent by end-users or malicious attackers gets through the gateway and impacts the backends behind. Our recommendation is to let the gateway know which query strings are in the API contract and specify them in the list, even when the list is long, and not use the wildcard. If the decision is to go with the wildcard, make sure your backends can handle abuse attempts from clients.

### Mandatory query string parameters
When your backend requires mandatory **query string** parameters and you want to make them **mandatory** in KrakenD, the only way to enforce this (without scripting) is using the `{variable}` placeholders in the endpoints definition. Mandatory means that the endpoint won't exist unless the parameter is passed. For instance:

```json
{
  "endpoint": "/v3/{channel}/foo",
  "backend": [
    {
            "host": ["http://backend"],
            "url_pattern": "/foo?channel={channel}"
    }
  ]
}
```

The parameter is mandatory as if a value for `channel` is not provided the server replies with a `404`.

With the configuration above a request to the KrakenD endpoint such as `http://krakend/v3/iOS/foo?limit=10&evil=here` makes a call to the backend with only the `channel` query string:

    /foo?channel=iOS

Nevertheless, the `input_query_strings` could also be added in this configuration, creating a special case of optional and mandatory parameters! You would be passing query strings both hardcoded in the `url_pattern` and generated from the user input. In this strange case, if the user passes a single optional query string parameter that is declared in `input_query_strings`, then the mandatory value is lost. The mandatory value is used if the request does not contain any known optional parameter. For instance:

```json
{
    "endpoint": "/v3/{channel}/foo",
    "input_query_strings": [
        "page",
        "limit"
    ],
    "backend": [
        {
            "host": [
                "http://backend"
            ],
            "url_pattern": "/foo?channel={channel}"
        }
    ]
}
```


With `http://krakend/v3/iOS/foo?limit=10&evil=here` the backend receives:

    /foo?limit=10

No mandatory `channel` here! Because the optional parameter `limit` has been declared.

On the other hand, `http://krakend/v3/iOS/foo?evil=here` produces:

    /foo?channel=iOS

No optional parameter has been passed, so the mandatory one is used.

Read the [`/__debug/` endpoint](/docs/v2.2/endpoints/debug-endpoint/) to understand how to test query string parameters.

## Headers forwarding
KrakenD **does not send client headers to the backend**, unless they are under the `input_headers` list. The list of headers sent by the client that you want to let pass to the backend must be written as an entry of the `input_headers` array (or there is an `"*"` entry).

**A client request from a browser or a mobile client contains a lot of headers**, including cookies. Typical examples of the variety of headers that clients send are `Host`, `Connection`, `Content-Type`,`Accept`, `Cache-Control`, `Cookie`... and a long, long, etcetera. Remember that unless explicitly defined, KrakenD won't let them pass. This security policy will save you from a lot of trouble.

### Default headers sent from KrakenD to Backends
KrakenD will act as an independent client connecting to your backends and will send this headers with its own values:

- `Accept-Encoding`
- `Host`
- `User-Agent` (KrakenD Version 2.2)
- `X-Forwarded-For`
- `X-Forwarded-Host`
- `X-Forwarded-Via` (only when `User-Agent` is in the `input_headers`)

In addition, when you use **tracing**, you might also see arrive **B3 propagation headers** in your backends, e.g.:

- `X-B3-Sampled`
- `X-B3-Spanid`
- `X-B3-Traceid`

### Overriding headers sent from KrakenD to Backends
When you use the `input_headers`, consider that any of the headers listed above are replaced with the ones you declare.

An example of passing the `User-Agent` to the backend:

```json
{
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/v1/foo",
      "input_headers": [
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
```

This setting changes the headers received by the backend to:

```yaml
Accept-Encoding: gzip
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
X-Forwarded-For: ::1
```

The `User-Agent` is no longer a KrakenD user-agent but a Mozilla one.

Read the [`/__debug/` endpoint](/docs/v2.2/endpoints/debug-endpoint/) to understand how to test headers.

### Sending all client headers to the backends
While the default policy prevents forwarding unrecognized headers, setting an asterisk `*` as the parameter name makes the gateway to **forward any header to the backends**, including cookies:

```json
{
  "endpoint": "/foo",
  "input_headers":[
      "*"
  ]
}
```

Enabling the wildcard **pollutes your backends**, as any header sent by end-users or malicious attackers gets through the gateway and impacts the backends behind (a famous exploit is the Log4J vulnerability). We recommend letting the gateway know which headers are in the API contract and specify them in the list, even when the list is long try to not use the wildcard. If the decision is to go with the wildcard, make sure your backends can handle abuse attempts from clients.

## Cookies forwarding
A cookie is just some content passing inside the `Cookie` header. If you want cookies to reach your backend, add the `Cookie` header under `input_headers`, just as you would do with any other header.

When doing this, **all your cookies** are sent to all backends inside the endpoint. Use this option wisely!

Example:

```json
{
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/v1/foo",
      "input_headers": [
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
```
