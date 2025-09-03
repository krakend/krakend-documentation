---
lastmod: 2022-12-13
old_version: true
date: 2019-10-02
linktitle: Strategies to return headers and errors
title: Returning the backend headers and errors
description: Implement returning of backend headers and errors to provide informative and customized error messages to API consumers
weight: 430
menu:
  community_v2.10:
    parent: "060 Request and Response Manipulation"
meta:
  #noop_incompatible: true
  since: v0.8
  source: https://github.com/luraproject/lura
  namespace:
  - backend/http
  scope:
  - backend
---

KrakenD's default policy regarding errors and status codes is to **hide from the client any backend details**, this includes headers and errors, except when you use the [`no-op` encoding](/docs/v2.10/endpoints/no-op/).

The philosophy behind this is that **clients have to be decoupled** from their underlying services, as an API Gateway should do. The opposite is a reverse proxy or a simple router.

## Strategies to return headers and errors
We do not recommend you to change the default behavior to have a secure and decoupled gateway. Yet, KrakenD provides flexibility so you can **override** the default policy of returning backend error details with different **strategies**.

These are the different strategies you can set:

- **Default strategy - Graceful degradation of the response** (when using something different than`no-op`): Supports multiple backends ([aggregation](/docs/v2.10/endpoints/response-manipulation/)). HTTP status codes returned to the client are essentially `200` or `500`, regardless of the backend(s) status codes. The body is built, merged, and manipulated from the working backends. The status codes of backend errors are logged in the console.
- **Headers and errors entirely handled by the backend** (default strategy for `no-op`): Shows the HTTP status codes, headers, and body as returned by the backend (**maximum one**). You cannot manipulate data (except with plugins or Lua). This option is mostly a reverse proxy, and its usage is discouraged. Use [`no-op` encoding](/docs/v2.10/endpoints/no-op/) when you want this.
- **Return the HTTP status code of a single backend**. Sets an empty body when there are errors (obfuscation), but preserves the HTTP status codes of the backend. Use the `return_error_code` flag in the backend ([see below](/docs/v2.10/backends/detailed-errors/#return-the-http-status-code-of-a-single-backend)).
- **Return the HTTP status code and error body of a single backend**. No obfuscation at all. Forwards the error body and the status code of a single backend. Use `return_error_code` and `return_error_msg` together (the latter is [declared as a `router` option](/docs/v2.10/service-settings/router-options/#return_error_msg) in the service level). The Content-Type from the backend is lost.
- **Return backend errors in a new key**. Supports multiple backends. Like the graceful degradation option, but it adds a new error key in the body when the backend fails -—ideal for debugging multiple backends. Use `return_error_details` (see below).
- **Show the error interpretation by the gateway but not the actual error body**. Semi-obfuscation. The client sees an interpretation of the gateway like *invalid status code*, or *context deadline exceeded* but does not see the actual error delivered by the backend. The status code is always 200 or 500. Use `return_error_msg` (at the [router level](/docs/v2.10/service-settings/router-options/#return_error_msg)). The flag is also compatible with `no-op`.

The different combinations are exemplified below.

## Return backend errors in a new key
If you prefer revealing error details to the client, you can show them in the gateway response. To achieve this, enable the `return_error_details` option in the `backend` configuration, and all errors will appear in the desired key.

Place the following configuration inside the `backend` configuration:

```json
{
    "url_pattern": "/return-my-errors",
    "extra_config": {
        "backend/http": {
            "return_error_details": "backend_alias"
        }
    }
}
```

The `return_error_details` option sets an alias for this backend. When a backend fails, you'll find an object named `error_` + its `backend_alias` containing the detailed errors of the backend. If there are no errors, the key won't exist.

An error example is:

```json
{
    "error_backend_alias": {
        "http_status_code": 404,
        "http_body": "404 page not found\\n"
    }
}
```

{{< note title="All status are 200" type="note" >}}
When you use `return_error_details`, all status codes returned to the client are `200`. The client must parse the response for the presence of the `error_backend_alias` or any other key you have set to determine if there's a problem.
{{< /note >}}


### Example
The following configuration sets an endpoint with two backends that return its errors in two different keys:

```json
{
        "endpoint": "/detail_error",
        "backend": [
            {
                "host": ["http://127.0.0.1:8081"],
                "url_pattern": "/foo",
                "extra_config": {
                    "backend/http": {
                        "return_error_details": "backend_a"
                    }
                }
            },
            {
                "host": ["http://127.0.0.1:8081"],
                "url_pattern": "/bar",
                "extra_config": {
                    "backend/http": {
                        "return_error_details": "backend_b"
                    }
                }
            }
        ]
    }
```

Let's say your `backend_b` has failed, but your `backend_a` worked just fine. The client's response could look like this:

```json
{
    "error_backend_b": {
        "http_status_code": 404,
        "http_body": "404 page not found\\n"
    },
    "foo": 42
}
```

## Return the HTTP status code of a single backend
When you have **one backend only** and use an encoding different than `no-op`, you can choose to return the original HTTP status code to the client.

The gateway obfuscates the HTTP status codes of the backend by default for many reasons, including security and consistency. Still, if you prefer using the HTTP status code of the backend instead, you can enable the `return_error_code` flag.

The body of the error will be empty unless you complement it with the `return_error_msg` at the `router` configuration (see below):

Place the following configuration in the configuration:

```json
{
  "version": 3,
  "$schema": "http://www.krakend.io/schema/krakend.json",
  "extra_config": {
    "router": {
      "return_error_msg": true
    }
  },
  "endpoints": [
    {
      "endpoint": "/return-status-and-error",
      "backend": [
        {
          "url_pattern": "/404",
          "host": [
            "http://somehost"
          ],
          "extra_config": {
            "backend/http": {
              "return_error_code": true
            }
          }
        }
      ]
    }
  ]
}
```

Notice that the `return_error_code` and the `return_error_details` are mutually exclusive. You can use one or the other but not both. If you declare them together, the gateway will use only `return_error_details`.


## Show an interpretation but not the error body
When you want to show the interpretation of the error but not the error of the backend, use the [router option `return_error_msg`](/docs/v2.10/service-settings/router-options/) as follows:

```json
{
  "version": 3,
  "$schema": "http://www.krakend.io/schema/krakend.json",
  "extra_config": {
    "router": {
      "return_error_msg": true
    }
  }
}
```