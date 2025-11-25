---
lastmod: 2025-05-09
date: 2018-07-20
aliases: ["/docs/features/parameter-forwarding/"]
title: Parameter Forwarding
linktitle: Forwarding query strings and headers
description: Learn how to forward and manipulate parameters effectively using KrakenD API Gateway, ensuring seamless communication between clients and microservices
weight: 30
menu:
  community_current:
    parent: "040 Routing and Forwarding"
---
KrakenD is an API Gateway with a **[zero-trust security policy](/docs/design/zero-trust/)**. You need to define what is allowed because KrakenD **does not forward** any unexpected [query string](#query-string-forwarding), [headers](#headers-forwarding), or [cookies](#cookies-forwarding). See below for instructions on how to set the forwarding rules.

![Diagram about parameters don't passing to backend](/images/documentation/diagrams/parameter-forwarding-1.mmd.svg)

## Configuration to enable parameter forwarding
You can change the default behavior according to your needs and define which elements can pass from the client to your backends. To do that, add the following configuration options under your `endpoint` definition:

{{< schema data="endpoint.json" filter="input_headers,input_query_strings" >}}

### Case-sensitive and case-insensitive parameters

- The `input_query_strings` list is **case sensitive**, as per the RFC specification. For instance, a request `?Page=1` and `?page=1` are considered different parameters, and only the latter will pass when setting `"input_query_strings": ["page"]`. If you expect multiple cases, add them all.
- The `input_headers` is **case-insensitive**, as per its RFC specification. It allows the passing of user headers in uppercase, lowercase, or mixed. Nevertheless, when the header is forwarded to the backend or used in other components, they receive it normalized in the **canonical format of the MIME header**, so users can mix capitalization and yet receive a consistent format.

{{< note title="Canonical Headers" type="info" >}}
While the `input_headers` declaration does not care about how you write the header (upper/lowercase), and KrakenD will access either way, when accessing or checking a header name through any components in KrakenD, you must write its canonical form regardless of what's being provided by the user.

The canonicalization **converts the first letter and any letter following a hyphen to uppercase**; the rest are converted to lowercase. For example, the canonical key for `accept-encoding`, `ACCEPT-ENCODING`, or `ACCept-enCODING` is `Accept-Encoding`. MIME header keys are assumed to be ASCII only. If the header contains a space or invalid header field bytes, it is returned without modifications.

If you get used to writing headers in the canonical format, you will save yourself from a lot of trouble.
{{< /note >}}


**Example**:

Send the query strings `items` and `page` to the backend, as well as `User-Agent` and `Accept` headers:

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
            "http://some.example.com:9000"
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

Add the **list** `input_query_strings` in your `endpoint` definition to enable the transition of query strings to your backend. For instance, let's forward `?items=10&page=2` to the backends now:

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
            "http://some.example.com:9000"
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

Also, if a request like `http://krakend:8080/v1/foo?items=10` does not include `page`, this parameter is simply missing in the backend request.

By definition, query string parameters are always optional, and the user can pass a subset of them, all or none. Suppose you want to enforce that the user provides a query string parameter. In that case, you must validate it with the [Common Expression Language](/docs/endpoints/common-expression-language-cel/) (faster) or with a [Lua script](/docs/endpoints/lua/) (slower).

### Sending all query string parameters
While the default policy prevents sending unrecognized query string parameters, setting an asterisk `*` as the parameter name makes the gateway to **forward any query string to the backends**:

```json
{
  "endpoint": "/foo",
  "input_query_strings":[
      "*"
  ]
}
```

**Enabling the wildcard pollutes your backends**, as any query string sent by end-users or malicious attackers gets through the gateway and impacts the backends behind. We recommend letting the gateway know which query strings are in the API contract and specify them in the list, even when it is long, and not use the wildcard. If you decide to go with the wildcard, ensure your backends can handle client abuse attempts.

### Mandatory query string parameters
When your backend requires mandatory **query string** parameters and you want to make them **mandatory** in KrakenD, the only way to enforce this with the open source edition (without scripting) is using the `{variable}` placeholders in the endpoints definition. Enterprise users can use [Dynamic Routing](/docs/enterprise/endpoints/dynamic-routing/#routing-based-on-query-strings) and [Security Policies](/docs/enterprise/security-policies/) together. Mandatory means that the endpoint won't exist unless the parameter is passed. For instance:

```json
{
  "endpoint": "/v3/{channel}/foo",
  "backend": [
    {
            "host": ["http://example.com"],
            "url_pattern": "/foo?channel={channel}"
    }
  ]
}
```

The parameter is compulsory; if a value for `channel` is not provided, the server replies with a `404`.

With the configuration above, a request to the KrakenD endpoint such as `http://krakend/v3/iOS/foo?limit=10&evil=here` makes a call to the backend with only the `channel` query string:

    /foo?channel=iOS

Nevertheless, the `input_query_strings` could also be added in this configuration, creating a special case of optional and mandatory parameters! You would pass query strings hardcoded in the `url_pattern` and generated from the user input. In this strange case, the mandatory value is lost if the user passes a single optional query string parameter that is declared in `input_query_strings`. The compulsory value is used if the request contains no known optional parameter. For instance:

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
                "http://example.com"
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

Read the [`/__debug/` endpoint](/docs/endpoints/debug-endpoint/) to understand how to test query string parameters.

## Headers forwarding
KrakenD **does not send client headers to the backend** except for the `Content-Type` unless they are under the `input_headers` list. The headers sent by the client that you want to let pass to the backend must be written explicitly in the `input_headers`. See below how to forward [all client headers](/docs/endpoints/parameter-forwarding/#sending-all-client-headers-to-the-backends) (and why it is a bad idea).

**A client request from a browser or a mobile client contains a lot of headers**, including cookies. Typical examples of the variety of headers clients send are `Host`, `Connection`, `Content-Type`, `Accept`, `Cache-Control`, `Cookie`... and a long etcetera. Remember that unless explicitly defined, KrakenD will only allow the `Content-Type`. This security policy will save you from a lot of trouble.


### Default headers sent from KrakenD to Backends
KrakenD will act as an independent client connecting to your backends and sending headers to them. Some are customizable, and others aren't.

#### Non-customizable headers
You cannot override the values of these headers (unless you have a plugin or a Lua script):

- `Host`
- `X-Forwarded-For`
- `X-Forwarded-Host`
- `X-Forwarded-Via` (only when `User-Agent` is in the `input_headers`)

The `X-Forwarded`-like headers are controlled by [`forwarded_by_client_ip`](/docs/service-settings/router-options/#forwarded_by_client_ip). All these headers **are always sent, regardless of whether you define them or not** under `input_headers`. You can override their values.

#### Customizable headers
The following headers are sent to the backends with default values, but if you add them under `input_headers`, you allow the client to send their own values. If the headers are under `input_headers`, but the client does not send them, KrakenD sets its values. The headers are:

- `Content-Type`, the one passed in the request (if passed)
- `Accept-Encoding`, set by KrakenD to `gzip`, unless you add it to the `input_headers`
- `User-Agent` Contains the value `KrakenD Version {{< product latest_version >}}`, unless you add it to the `input_headers`

Needless to say that you can add any other header that is not listed in this page under `input_headers` and KrakenD will forward it to the backend.

In addition, there are a few KrakenD components that support **setting attributes in headers**, like for instance [`propagate_claims` in JWT](/docs/authorization/jwt-validation/#propagate_claims). These components will send the additional headers you configure to the backend automatically.

### Overriding headers sent from KrakenD to Backends
When you use the `input_headers`, consider that any headers listed above are replaced with the ones you declare.

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
            "http://some.example.com:9000"
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

The `User-Agent` is no longer a KrakenD user agent but a Mozilla one.

Read the [`/__debug/` endpoint](/docs/endpoints/debug-endpoint/) to understand how to test headers.

### Sending all client headers to the backends
While the default policy prevents forwarding unrecognized headers, setting an asterisk `*` as the parameter name makes the gateway to **forward any header to the backends**, **including cookies**:

```json
{
  "endpoint": "/foo",
  "input_headers":[
      "*"
  ]
}
```

Enabling the wildcard **pollutes your backends**, as any header sent by end-users or malicious attackers gets through the gateway and impacts the backends behind (a famous exploit is the Log4J vulnerability). Let the gateway know which headers are in the API contract and specify them in the list. Even when the list is long, try not to use the wildcard. If the decision is to go with the wildcard, make sure your backends can handle client abuse attempts, and do not discard adding a second `input_headers` list in the backend (not all backends in aggregation might need every header).

{{< note title="Exception for X-Forwarded-like headers" type="warning" >}}
Even though you allow to pass all headers with `"input_headers": ["*"]`, you cannot override the headers `X-Forwarded-For`, `X-Forwarded-Host`, and `X-Forwarded-Via` which are automatically calculated by KrakenD.

If you'd like to take those headers into account, use the flag [`forwarded_by_client_ip` under the router section](/docs/service-settings/router-options/#forwarded_by_client_ip)
{{< /note >}}


### Granular header filtering
All headers listed in the `input_headers` parameter hit every single backend of the `endpoint`. If you want to add a second level of filtering, you can configure the `input_headers` list in the `backend` section too. Doing this allows you to have backends that receive fewer headers than other backends in the same endpoint.

For instance, the following endpoint allows passing two headers to its backends, but the second backend allows a single header to pass:

```json
{
  "version": 3,
  "host": [
    "http://some.example.com:9000"
  ],
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
          "url_pattern": "/receive-defined-headers",
        },
        {
          "url_pattern": "/receive-one-header-only",
          "input_headers": [
            "User-Agent"
          ]
        }
      ]
    }
  ]
}
```

## Cookies forwarding
A cookie is just some content passing through the `Cookie` header. If you want cookies to reach your backend, add the `Cookie` header under `input_headers`, just as you would with any other header.

**All your cookies** are sent to all backends inside the endpoint when doing this. Use this option wisely!

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
            "http://some.example.com:9000"
          ]
        }
      ]
    }
  ]
}
```
