---
lastmod: 2021-01-28
old_version: true
date: 2020-02-26
linktitle:  Creating endpoints
title: Endpoint Configuration in KrakenD API Gateway
description: Learn how to configure and manage endpoints effectively in KrakenD API Gateway, enabling seamless integration and orchestration of microservices.
weight: -10
menu:
  community_v1.4:
    parent: "040 Endpoint Configuration"
---
KrakenD endpoints are the essential part of KrakenD as they are what your end users consume. 

See [Understanding the configuration file](/docs/v1.4/configuration/structure/) if you haven't read it yet.

To create an endpoint you only need to add an **endpoint object** under the `endpoints` list with the resource you want to expose. If no `method` is declared, it's assumed to be read-only (`GET`).

The endpoints section looks like this:

    "endpoints": [
    {
      "endpoint": "/v1/foo",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/bar",
          "method": "GET",
          "host": [
            "https://api.mybackend.com"
          ]
        }
      ]
    }
    ]

The previous example exposes to the clients a `GET /v1/foo` and takes the data from your existing backend `https://api.mybackend.com"/bar`. As there is no other additional configuration, the data won't be manipulated. 

## Attributes
The **endpoint object** accepts the following attributes:

- `endpoint`: The resource URL you want to expose
- `method`: Must be **written in uppercase** `GET`, `POST`, `PUT`, `PATCH`, `DELETE`.
- `backend`: List of all the **backend objects** queried for this endpoint. 
- `extra_config`: Configuration of components and middlewares that are executed with this endpoint.
- `querystring_params`: Recognized GET parameters. See [parameter forwarding](/docs/v1.4/endpoints/parameter-forwarding/).
- `headers_to_pass`: Forwarded headers. See [headers forwarding](/docs/v1.4/endpoints/parameter-forwarding/#headers-forwarding).
- `concurrent_calls`: A technique to improve response times. See [concurrent requests](/docs/v1.4/endpoints/concurrent-requests/)
- `cache_ttl`: (*time unit*) The cache headers informing for how long the CDN can cache the request to this endpoint. Related: [caching backend responses](/docs/v1.4/backends/caching/).
- `timeout`: (*time unit*) Maximum time you'll wait for the slowest backend response. Usually specified in seconds (`s`) or milliseconds (`ms`. E.g: `1500ms`)

\* Valid _time units_ are: `ns`, `us`, (or `µs`), `ms`, `s`, `m`, `h` E.g: `1s`


### Multiple methods of the same resource

The `method` key defines the HTTP verb you can use with the endpoint. You need to declare **one endpoint object for each method**. So if you want the same endpoint to listen to `GET` and `POST` requests you need the following configuration:

    "endpoints": [
    
    {
      "endpoint": "/v1/foo",
      "backend": [
        {
          "url_pattern": "/bar",
          "host": [
            "https://api.mybackend.com"
          ]
        }
      ]
    },

    {
      "endpoint": "/v1/foo",
      "method": "POST",
      "backend": [
        {
          "url_pattern": "/bar",
          "method": "POST",
          "host": [
            "https://api.mybackend.com"
          ]
        }
      ]
    }

    ]

Notice that the `method` is declared both in the endpoint and in the backend (as they could be different). 

### Endpoint variables

Endpoints can define variables in its endpoint definition. To do so, encapsulate the variable name with curly braces, like `{var}`. 

    {
      "endpoint": "/user/{id}",
      ...
    }

The previous endpoint will accept requests like `/user/123` or `/user/A-B-C` or anything else while it does not contain another slash. A request like `/user/1/2` won't be recognized by this endpoint. For such scenario you need to declare `/user/{first}/{second}`.

### Colliding routes

KrakenD router is based on httprouter, offering a brutal performance. Nevertheless this design decision comes with the following considerations:

- You cannot create conflicting routes
- Since this router has only explicit matches, you can not register static routes and variables for the same path segment. For example you can not register the patterns `/user/new` and `/user/{id}` for the same request method at the same time. The routing of different request methods is independent from each other.

Declaring a colliding route will result in a panic during startup, such as:

    panic: wildcard route ':id' conflicts with existing children in path '/user/:id'

### Other limitations
URLs do not support colons `:` in the definition.
