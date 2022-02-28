---
lastmod: 2021-12-09
date: 2018-07-20
aliases: ["/docs/features/parameter-forwarding/"]
linktitle:  Query strings and headers
title: Forwarding query strings and headers
weight: 10
menu:
  community_current:
    parent: "040 Endpoint Configuration"
---
KrakenD is an API Gateway with a **zero-trust policy**, and when it comes to forward query strings, cookies, and headers there are implications that you need to be aware of.

Part of the zero-trust policy implies that KrakenD **removes by default** any unexpected [query string](#query-string-forwarding), [headers](#headers-forwarding), or [cookies](#cookies-forwarding). Not trusting anyone by default means that unless otherwise configured, none of these elements are forwarded to or included in your API backends.

## Configuration to enable parameter forwarding
You can change the default behavior according to your needs, and define which elements are allowed to pass. To do that, add the following configuration options under your `endpoint` definition:

- `input_query_strings` (*array*): Defines the exact list of query strings that are allowed to reach the backend when passed
- `input_headers` (*array*): Defines the list of all headers allowed to reach the backend when passed
- A single *star* element (`["*"]`) as the options above, forwards **everything** to the backend

{{< note title="Case sensitive parameters" type="info" >}}
The `input_query_strings` and `input_headers` lists are **case sensitive**. For instance, a request `?Page=1` wont passed to the backend with `"input_query_strings": ["page"]`
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
The zero-trust policy implies that for instance, if a KrakenD endpoint `/foo` receives the request `/foo?items=10&page=2`, all its declared backends are not going to see neither `items` nor `page`, **unless otherwise configured**.

To enable the transition of query strings to your backend, add the **list** `input_query_strings` in your `endpoint` definition. For instance, let's forward `?items=10&page=2` to the backends now:

{{< highlight json >}}
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
{{< /highlight >}}

The `input_query_strings` list has the following behavior:

- **Items in the list** are forwarded to your backend when passed
- **Additional query strings not in the list** are removed from the final call
- **Writing a single *star* element** (`"input_query_strings":["*"]`) instead of individual strings, forwards **everything** to the backend



Parameters are always optional and the user can pass a subset of them, all, or none. If you want to enforce that a query string parameter is passed by the user, you will need to do a validation with the Common Expression Langiu




With this configuration, given a request like `http://krakend:8080/v1/foo?items=10&page=2&evil=here`, the backend receives `items` and `page`, but `evil` is missing.

Also, if a request like `http://krakend:8080/v1/foo?items=10` does not include `page`, this parameter is simply missing in the backend request as well.

### Sending all query string parameters
While the default policy prevents from sending unrecognized query string parameters, setting an asterisk `*` as the parameter name makes the gateway to **forward any query string to the backends**:

{{< highlight js >}}
  "input_query_strings":[  
        "*"
  ]
{{< /highlight >}}

**Enabling the wildcard pollutes your backends**, as any query string sent by end users or malicious attackers gets through the gateway and impacts the backends behind. Our recommendation is to let the gateway know which are the query strings in the API contract and specify them in the list, even when the list is long, and not use the wildcard. If the decision is to go with the wildcard, make sure your backends can handle abuse attempts from clients.

### Mandatory query string parameters
When your backend requires mandatory **query string** parameters and you want to make them **mandatory** in KrakenD, the only way to enforce this is using the `{variable}` placeholders in the endpoints definition. Mandatory means that the endpoint won't exist unless the parameter is passed. For instance:

{{< highlight json >}}
{
  "endpoint": "/v3/{channel}/foo",
  "backend": [
    {
            "host": ["http://backend"],
            "url_pattern": "/foo?channel={channel}"
    }
  ]
}
{{< /highlight >}}


The parameter is mandatory as if a value for `channel` is not provided the server replies with a `404`.

With the configuration above a request to the KrakenD endpoint such as `http://krakend/v3/iOS/foo?limit=10&evil=here` makes a call to the backend with only the `channel` query string:

    /foo?channel=iOS

Nevertheless, the `input_query_strings` could also be added in this configuration, creating a special case of optional and mandatory parameters! You would be passing query strings both hardcoded in the `url_pattern` and generated from the user input. What happens in this strange case is that if the user passes a single optional query string parameter that is declared in `input_query_strings` then the mandatory value is lost. If the request does not contain any known optional parameter, then the mandatory value is used. For instance:

{{< highlight json >}}
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
{{< /highlight >}}


With `http://krakend/v3/iOS/foo?limit=10&evil=here` the backend receives:

    /foo?limit=10

No mandatory `channel` here! Because the optional parameter `limit` has been declared.

On the other hand, `http://krakend/v3/iOS/foo?evil=here` produces:

	 /foo?channel=iOS

No optional parameter has been passed, so the mandatory one is used.

Read the [`/__debug/` endpoint](/docs/endpoints/debug-endpoint/) to understand how to test query string parameters.

## Headers forwarding
KrakenD **does not send client headers to the backend by default**.  Use `input_headers`.

Declare the list of headers sent by the client that you want to let pass to the backend with the `input_headers` option.

A client request from a browser or a mobile client usually contains a lot of headers, including cookies. Typical examples of the variety of headers that clients send are `Host`, `Connection`, `Cache-Control`, `Cookie`... and a long, long etcetera. The backend usually does not need any of this to return the content.

KrakenD passes only these essential headers to the backends:

- `Accept-Encoding`
- `Host`
- `User-Agent` (KrakenD Version {{< version >}})
- `X-Forwarded-For`
- `X-Forwarded-Host`
- `X-Forwarded-Via` (only when `User-Agent` is in the `input_headers`)

When you use the `input_headers`, take into account that any of these headers are replaced with the ones you declare.

 An example to pass the `User-Agent` to the backend:

{{< highlight json >}}
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
{{< /highlight >}}


This setting changes the headers received by the backend to:

{{< highlight yaml >}}
Accept-Encoding: gzip
Host: localhost:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
X-Forwarded-For: ::1
{{< /highlight >}}

Read the [`/__debug/` endpoint](/docs/endpoints/debug-endpoint/) to understand how to test headers.

### Sending all client headers to the backends
While the default policy prevents forwarding unrecognized headers, setting an asterisk `*` as the parameter name makes the gateway to **forward any header to the backends**, including cookies:

{{< highlight js >}}
"input_headers":[  
      "*"
]
{{< /highlight >}}

Enabling the wildcard pollutes your backends, as any header sent by end users or malicious attackers gets through the gateway and impacts the backends behind. Our recommendation is to let the gateway know which are the headers in the API contract and specify them in the list, even when the list is long, and not use the wildcard. If the decision is to go with the wildcard, make sure your backends can handle abuse attempts from clients.


## Cookies forwarding
A cookie is just some content passing inside the `Cookie` header. If you want cookies to reach your backend, add the `Cookie` header under `input_headers`, just as you would do with any other header.

When doing this, **all your cookies** are sent to all backends inside the endpoint. Use this option wisely!

Example:

{{< highlight json >}}
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
{{< /highlight >}}
