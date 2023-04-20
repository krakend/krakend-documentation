---
lastmod: 2022-12-13
old_version: true
date: 2019-10-02
linktitle: Return backend errors
title: Strategies to return the backend errors
weight: 130
menu:
  community_v2.2:
    parent: "050 Backends Configuration"
meta:
  #noop_incompatible: true
  since: 0.8
  source: https://github.com/luraproject/lura
  namespace:
  - backend/http
  scope:
  - backend
---

When you are willing to manipulate or aggregate data, KrakenD's default policy regarding errors and status codes is to **hide from the client any backend details**, except when you use the [`no-op` encoding](/docs/v2.2/endpoints/no-op/). The philosophy behind this is that clients have to be decoupled from their underlying services.

You can override the default policy of returning backend error details with different strategies:

- **Errors entirely handled by the backend** (default strategy for `no-op`): Show the HTTP status codes, headers, and body as returned by the backend (maximum one). You cannot manipulate data (except with plugins). Use [`no-op` encoding](/docs/v2.2/endpoints/no-op/).
- **Graceful degradation of the response** (default strategy for non-`no-op`): HTTP status codes are essentially 200 or 500, regardless of the backend(s) status codes. The body is built, merged, and manipulated from the working backends.
- **Include the backend errors in a new key**. Same as above, but it shows the errors in a new key in the body—ideal for debugging multiple backends. Use `return_error_details`.
- **Forward the HTTP status code of a single backend**. Sets an empty body when there are errors (obfuscation), but preserves the HTTP status codes of the backend. Use `return_error_code`.
- **Forward the HTTP status code and error body**. No obfuscation. Forwards the error body and the status code of a single backend. Use `return_error_code` and `return_error_msg` together.
- **Show an interpretation but not the error body**. Semi-obfuscation. The client sees an interpretation of the gateway like *invalid status code*, or *context deadline exceeded* but does not see the actual error delivered by the backend. The status code is always 200 or 500. Use `return_error_msg`. The flag is also compatible with `no-op`.

The different combinations are exemplified below.

## Include the backend errors in a new key
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

## Forward the HTTP status code of a single backend
When you have **one backend only** and use an encoding different than `no-op`, you can choose to return the original HTTP status code to the client.

The gateway obfuscates the HTTP status codes of the backend by default for many reasons, including security and consistency. Still, if you prefer using the HTTP status code of the backend instead, you can enable the `return_error_code` flag.

The body of the error will be empty unless you complement it with the `return_error_msg` at the `router` configuration (see below):

Place the following configuration in the configuration:

```json
{
  "version": 3,
  "$schema": "http://www.krakend.io/schema/v3.json",
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
When you want to show the interpretation of the error but not the error of the backend, use the [router option `return_error_msg`](/docs/v2.2/service-settings/router-options/) as follows:

```json
{
  "version": 3,
  "$schema": "http://www.krakend.io/schema/v3.json",
  "extra_config": {
    "router": {
      "return_error_msg": true
    }
  }
}
```