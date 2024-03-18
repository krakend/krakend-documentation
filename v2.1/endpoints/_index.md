---
lastmod: 2022-10-11
old_version: true
date: 2020-02-26
linktitle:  The endpoint object
title: Endpoint Configuration
description: Learn how to configure and manage endpoints effectively in KrakenD API Gateway, enabling seamless integration and orchestration of microservices.
weight: -10
menu:
  community_v2.1:
    parent: "040 Endpoint Configuration"
---
KrakenD `endpoints` are the most critical configuration part of KrakenD, as they are what your end users consume. Adding endpoint objects creates the API contract your users will consume.

{{< note title="Configuration overview" type="tip" >}}
If you are still unfamiliar with KrakenD's configuration structure, take a moment to read [Understanding the configuration file](/docs/v2.1/configuration/structure/).
{{< /note >}}

The `endpoints` array contains the **API definition you are publishing**. It is a collection of **endpoint objects**, and you have to place it at the root of your configuration file.


## The endpoint object

To create an endpoint, you only need to add an **endpoint object** under the `endpoints` collection. An endpoint object should contain at least the `endpoint` name and a `backend` section (to where it connects to). The defaults are taken if no further information is declared (e.g.: `method` will be a `GET`, and `output_encoding` as `json`).

An `endpoints` section might look like this:

```json
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
            "https://api.mybackend.com"
          ]
        }
      ]
    }
  ]
}
```


The previous example exposes to the clients a `GET /v1/users/{user}` endpoint and takes the data from your existing backend `GET https://api.mybackend.com/users/summary/{user}`.

Inside this object, you can add manipulation options and transform the response before it returns to the end user.

### Endpoint configuration
The configuration attributes of endpoints are:
{{< schema version="v2.1" data="endpoint.json" >}}

### Endpoints with multiple nesting levels
You might have envisioned KrakenD as a proxy and expected its `endpoint` declaration to **work as a prefix** and listen to any path with an undetermined number of nesting levels. **But KrakenD does not work like this by default**. Instead, it expects you to declare every possible URL structure.

For instance, you declared an `"endpoint": "/user/{id}"` and you expected to resolve URLs like `/user/john/profile/preferences`, but you are getting a *404* instead. There are two solutions to this problem:

1. You can declare all possible endpoints: `/user/{id}`, `/user/{id}/{level2}`, `/user/{id}/{level2}/{level3}`, etc.
2. You use a [Wildcard {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/v2.1/endpoints/wildcard/)


### Endpoints listening to multiple methods

The `method` attribute defines the HTTP verb you can use with the endpoint. If you need to support multiple methods (e.g.,  `GET`, `POST`, `DELETE`) in the same endpoint, you will need to declare **one endpoint object for each method**. So if you want the same endpoint to listen to `GET` and `POST` requests, you need the following configuration:

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
            "https://api.mybackend.com"
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
            "https://api.mybackend.com"
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
