---
lastmod: 2022-04-06
old_version: true
date: 2019-02-22
toc: true
linktitle: No-op (proxy only)
title: Proxying directly to the backends with `no-op`
weight: 40
menu:
  community_v2.11:
    parent: "040 Routing and Forwarding"
meta:
  since: v1.2
  source: https://github.com/krakend/krakend-ce
  namespace: false
  scope:
  - endpoint
---
KrakenD `no-op` (**no-operation**), is a special type of **encoding** that behaves as a **proxy** by passing the client's request to the backend and returning the response to the client ***as it is*** (some additional operations are permitted). Essentially without any manipulation or operation.

## Using `no-op` to proxy requests
When setting `no-op`, KrakenD does not inspect the request `body` or manipulates it in any way. Instead, when a request to a `no-op` endpoint is received, KrakenD directly forwards it to the backend without doing any operation with it.

The *proxy pipe* (this is from KrakenD to backend) is marked to do no-operation, meaning that KrakenD does not aggregate content, filter, manipulate or any of the other functionalities performed during this pipe. It's also important to notice that only a **single backend** is accepted, as the merge operation happens during the *proxy pipe*.

Employing the same principle, when the backend produces the response, it's passed back to the client *as is*, preserving its form: body, headers, status codes and such.

On the other hand, the *router pipe*'s features (from client to KrakenD) remain unaltered, meaning that for instance you can still rate-limit your end-users or require JWT authorization to name a few examples.

## Key concepts
The **key concepts** of `no-op` are:

- The KrakenD endpoint works just like a regular proxy
- The *router pipe* functionalities are available (e.g., rate limiting the endpoint)
- The *proxy pipe* functionalities are disabled (aggregate/merge, filter, manipulations, body inspection, concurrency...)
- Headers passing to the backend still need to be declared under `input_headers`, as they hit the router layer first.
- Query strings passing to the backend still need to be declared under `input_query_strings`, as they hit the router layer first.
- Backend response and headers remain unchanged (including status codes)
- The body cannot be changed and is set solely by the backend
- `1:1` relationship between endpoint-backend (one backend per endpoint).
- `X-Krakend-Completed` will be false and `X-Krakend` with current version will be added in response headers


## When to use `no-op`
Use `no-op` when you need to **couple the client with a specific backend without any KrakenD manipulation**.

Examples:

- You want to set a `Cookie` to the client directly from the backend.
- You need to keep the headers of the backend as is.


## How to use `no-op`
To declare endpoints that return the backend response as it is you need to define `"output_encoding": "no-op"`. KrakenD will set the `"encoding": "no-op"` in the `backend` section automatically, ignoring any different value you might have set.

When using the no-op encoding remember that the endpoint can only have **one backend** as KrakenD is not going to inspect or manipulate the response (no merging happens). Also, other pipe options like the concurrent requests, or manipulation options are not available: you will find them flagged across the documentation as not compatible with no-op.

## Example
The following snippet shows an endpoint that is passed to the backend as is. Notice that both the endpoint and the backend have a `no-op` encoding. The backend is using KrakenD's debug endpoint to capture the request in the console:

{{< highlight json "hl_lines=3 6" >}}
{
    "endpoint": "/auth/login",
    "output_encoding": "no-op",
    "backend": [
        {
            "encoding": "no-op",
            "host": [ "localhost:8080" ],
            "url_pattern": "/__debug/login"
        }
    ]
}
{{< /highlight >}}
