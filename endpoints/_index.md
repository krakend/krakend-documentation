---
lastmod: 2025-10-16
date: 2020-02-26
aliases: ["/docs/endpoints/creating-endpoints/"]
linktitle:  The endpoint object
title: Endpoint Configuration
description: Configure and manage API endpoints effectively with KrakenD Enterprise. Explore our documentation to learn how to define and optimize your API endpoints for better performance.
weight: 10
menu:
  community_current:
    parent: "040 Routing and Forwarding"
dark_header_image: true
images:
- /images/documentation/hero/endpoints.png
---
**Endpoints** are the most important core concept in KrakenD. An oversimplification is that they define the **available routes and protocols the gateway supports**. For instance, you create a `/foo` route and this connects to a `foo.example.com` to fetch the data. If you are familiar with the *API or AI Gateway offering*, as you continue reading, you will quickly realize that **KrakenD is quite different from other products**.

Instead of acting as a mere proxy and coupling one sad route to one sad underlying service of the same type, in KrakenD you define **the API contract** you want to expose, and then you create the composite of services that fulfill the contract. Said otherwise, create whatever API definition you want, and plug it into your existing stack. It does not matter the protocols or encodings you have on each side, as KrakenD takes care of transforming the data for you.

For example, say you declare a route that offers REST API content on `/foo, but you want to aggregate the results of querying a gRPC server, a regular REST API, a Lambda service, and an LLM Model altogether. Or maybe you want to offer all the latter under a gRPC server. Or maybe you have a twenty-year-old SOAP service and want to create an MPC Server instantly so your AI client can talk "naturally" to it. All that and more is available as an endpoint.

Because KrakenD treats users' interactions with the gateway and the gateway's interactions with services separate, endpoints are a piece of magic that can complete all kinds of use cases.

## The endpoint object
{{< note title="Deep dive into the configuration" type="tip" >}}
If you are still getting familiar with KrakenD's configuration structure, take a moment to read [Understanding the configuration file](/docs/configuration/structure/). This section explains what an `endpoint` looks like, but take a moment to understand the different pieces.
{{< /note >}}

We refer to all routes as families, regardless of their transport nature (HTTP, WebSockets, SOAP, etc.), as `endpoints`. Each of these endpoints can retrieve data from one or more different upstream services, which we collectively call the `backend` array.

The `endpoints` array in the configuration contains the **API definition you are publishing**, and is a collection of **endpoint objects** that granularly define how each route behaves. Because everything is independent and highly configurable, the gateway can feed from a vast ecosystem and can add new functionality to your stack.

To create an endpoint, you only need to add an **endpoint object** under the `endpoints` collection of the configuration. An endpoint object should contain at least the `endpoint` name and a `backend` section (to which it connects). The defaults are taken if no further information is declared (e.g., `method` will be a `GET`, and `output_encoding` as `json`).

A simple`endpoints` section for a REST endpoint might look like this:

```json
{
  "$schema": "https://www.krakend.io/schema/v{{< product minor_version >}}/krakend.json",
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/v1/users/{user}",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/users/summary/{user}",
          "method": "GET",
          "host": [
            "https://api.example.com"
          ]
        }
      ]
    }
  ]
}
```

The previous example exposes a REST `GET /v1/users/{user}` endpoint to the clients and takes the data from your existing backend `GET https://api.example.com/users/summary/{user}`.

This is the most simple example of an endpoint definition, from here you can add manipulation options, transform the request or the response, and a lot more stuff. See the options below.

### Endpoint object configuration
When declaring `endpoints` objects in the configuration, the following fields are available for each one of them:

{{< schema data="endpoint.json" >}}

### Endpoints with multiple nesting levels
You might have envisioned KrakenD as a proxy and expected its `endpoint` declaration to **work as a prefix** and listen to any path with an undetermined number of nesting levels. **But KrakenD does not work like this by default**. Instead, it expects you to declare every possible URL structure.

For instance, you declared an `"endpoint": "/user/{id}"` and you expected to resolve URLs like `/user/john/profile/preferences`, but you are getting a *404* instead. There are two solutions to this problem:

1. You can declare all possible endpoints: `/user/{id}`, `/user/{id}/{level2}`, `/user/{id}/{level2}/{level3}`, etc.
2. You use a [Wildcard](/docs/enterprise/endpoints/wildcard/)


### Endpoints listening to multiple methods

The `method` attribute defines the HTTP verb you can use with the endpoint. If you need to support multiple methods (e.g.,  `GET`, `POST`, `DELETE`) in the same endpoint, you must declare **one endpoint object for each method**. So if you want the same endpoint to listen to `GET` and `POST` requests, you need the following configuration:

{{< highlight json "hl_lines=4 5 9 17 18 22">}}
{
  "endpoints": [
    {
      "endpoint": "/v1/users/{user}",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/users/summary/{user}",
          "method": "GET",
          "host": [
            "https://api.example.com"
          ]
        }
      ]
    },
    {
      "endpoint": "/v1/users/{user}",
      "method": "POST",
      "backend": [
        {
          "url_pattern": "/users/summary/{user}",
          "method": "POST",
          "host": [
            "https://api.example.com"
          ]
        }
      ]
    }

  ]
}
{{< /highlight >}}
Notice that the `method` is declared both in the endpoint and in the backend (as they could be different).

### Endpoint variables

As you can see in the examples above, endpoints can define variables in their endpoint definition. To do so, encapsulate the variable name with curly braces, like `{var}`.

```json
{
  "endpoint": "/user/{id}"
}
```


The previous endpoint will accept requests like `/user/123` or `/user/A-B-C`. But **it won't take** a request like `/user/1/2`, as there is an extra slash than the definition, and KrakenD considers this to be a different endpoint.

### Router rules to avoid collisions
When you declare multiple endpoints that **share common prefixes**, make sure that you do not declare the same route with different variable names.

For instance, you cannot have the following two endpoints coexisting:

- endpoint: `/user/{userid}`
- endpoint: `/user/{iduser}/some`

This will cause a `panic` on startup (that you can catch earlier if you run a `krakend check -t -c krakend.json` )

But you can have the same routes declared as:

- endpoint: `/user/{userid}`
- endpoint: `/user/{userid}/some`

And this will work.

Similarly you can't do this:

- endpoint: `/v1/{domain}/user/{userid}`
- endpoint: `/v1/{domain}/user/{iduser}/some`

But you can declare the same route as:

- endpoint: `/v1/{domain}/user/{userid}`
- endpoint: `/v1/{domain}/user/{userid}/some`

Summarizing, on colliding routes, make sure to use the same variable names.


### Disable RESTful checking
By default KrakenD only works with **RESTful URL patterns** in its endpoint definition. Enable the option [`disable_rest`](/docs/service-settings/http-server-settings/#disable_rest) in the root of your configuration if need unrestful endpoints, e.g.: `/file.{extension}`

{{< highlight json "hl_lines=4 13">}}
{
  "$schema": "https://www.krakend.io/schema/v{{< product minor_version >}}/krakend.json",
  "version": 3,
  "disable_rest": true,
  "endpoints": [
    {
      "endpoint": "/foo/{var}/file.{extension}",
      "backend": [
        {
          "host": [
            "http://example.com"
          ],
          "url_pattern": "/{var}.{extension}"
        }
      ]
    }
  ]
}
{{< /highlight >}}

## Automatic protocol and encoding translation
The endpoints return HTTP content to the end-user in any of the [supported encodings](/docs/endpoints/content-types/), regardless of the type of backend you are connecting to.

If, for instance, one of the backends you are connecting to uses AMQP, Kafka, gRPC, or any other supported services, KrakenD will perform automatically for you both the protocol and the encoding translation.

## Additional endpoint functionality
The `extra_config` entry of any endpoint can include additional components that provide further functionality. The following is the list of all components accepted under `extra_config`, see the linked documentation of each for more information:

{{< schema data="endpoint_extra_config.json" norecurse="auth/api-keys,proxy" title="Available components of an endpoint">}}
