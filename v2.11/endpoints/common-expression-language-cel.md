---
lastmod: 2025-04-10
old_version: true
date: 2019-01-24
linktitle: "Conditional requests and responses"
title: Conditional requests and responses with CEL
description: Learn how to utilize the Common Expression Language (CEL) in KrakenD API Gateway for dynamic routing and advanced request handling
weight: 180
images: ["/images/documentation/krakend-cel.png"]
dark_header_image: true
menu:
  community_v2.11:
    parent: "040 Routing and Forwarding"
meta:
  since: v0.8
  source: https://github.com/krakend/krakend-cel
  namespace:
  - validation/cel
  scope:
  - endpoint
  - backend
  - async_agent
  log_prefix:
  - "[ENDPOINT: /foo][CEL]"
  - "[BACKEND: /foo][CEL]"
---
There are times when you might want to incorporate **additional logic** to check if the gateway has to **skip the backend call**. For example, maybe the request from the user is undoubtedly wrong, and there is no point in hitting your backend(s).

There are other times that you might need to **skip returning the response** because after parsing it you realize that it is not worth showing it to the user, but rather return an error.

In both scenarios where you check requests and responses, the **Common Expression Language (CEL)** implements standard semantics for expression evaluation and is a straightforward and powerful option to have complete control during requests and responses.

When the CEL component is enabled, you can set any number of expressions to check both requests and responses, either at the endpoint or backend level. **CEL does not transform the data, but it gives you the control of deciding what to do in the next step.**

## How CEL works
In any `endpoint`, `backend`, or `async_agent`, you can define a sequence of expressions you'd like to check using [Google's CEL spec](https://github.com/google/cel-spec) to write the conditions.

During runtime, when an expression returns `false`, KrakenD aborts the execution of that layer and **logs an error** like `request aborted by evaluator #0` (the number of CEL expression that failed). When it fails, it does not return the content or it does not perform the request (depending on the type). Otherwise, KrakenD serves the content if all expressions return `true`.

The CEL expressions will sound familiar if you are used to languages like javascript, C, C++, or Java to name a few. The expressions need to represent a boolean condition. For instance:

```js
'::1' in req_headers['X-Forwarded-For']
```

This expression checks that the request comes from localhost by checking that the header array `X-Forwarded-For`. In this example `::1` is the loopback address for IPv6 (`127.0.0.1` in IPv4)

## Types of CEL evaluations
You can use CEL expressions in five different places: during the **request** and the **response** of both **backends** and **endpoints** (see the blue dots in the image), and prior to the endpoint call when used as a JWT rejecter. The flow is:

![The 5 CEL places of action](/images/documentation/krakend-cel.png)

- **JWT** (token) evaluation (**Note:** you must include the `auth/validator`)
- **Endpoint request** evaluation
- **Backend request** evaluation (per N backends)
- **Backend response** evaluation (per N backends)
- **Endpoint response** evaluation (can evaluate all merged data)

## Configuration
The CEL component goes inside the `extra_config` of your `endpoints` or your `backend` using the namespace `validation/cel`.

Depending on where you put the `extra_config`, the gateway will check the expressions at the `endpoint` level, or the `backend` level.

For instance, you might want to reject users that do not adhere to some criteria related to the content in their JWT token. There is no reason to delay this check, and you would place the examination at the endpoint level right before hitting any backend. In another scenario, you might want to ensure that the response of a specific backend contains a must-have field; that configuration would go under the `backend` section and be isolated from the rest of sibling backends under the same endpoint umbrella.

Finally, when combined with the [sequential proxy](/docs/v2.11/endpoints/sequential-proxy/), you can skip requesting a backend if a previous call didn't fulfill your criteria.

The configuration is as follows:

```json
{
    "extra_config": {
        "validation/cel": [
            {
                "check_expr": "CONDITION1 && CONDITION2"
            },
            {
                "check_expr": "CONDITION3 && CONDITION4"
            }
        ]
    }
}
```
**Notice that the CEL object is an array**, even when you need a single evaluation object. If **all stacked conditions** in the array are *true*, the request/response continues. As soon as it finds a *false*, the validation fails.

Each object in the array has the following syntax:

{{< schema version="v2.11" data="validation/cel.json" property="items" >}}

See the sections below to use **additional variables**.
{{< note title="A note on client headers" >}}
When **client headers** are needed, remember to add them under [`input_headers`](/docs/v2.11/endpoints/parameter-forwarding/#headers-forwarding) as KrakenD does not forward headers to the backends unless declared in the list.
{{< /note >}}


## Adding logic in the requests and responses.
There are three different ways to access the metadata of requests and responses when you are inside the `check_expr` to decide whether or not to continue serving the user command.

- Use a `req_` type variable to access **request** data.
- Use a `resp_` type variable to access **response** data.
- Use the `JWT` variable to access the **payload of the JWT** (requires `auth/validator` and being in the `endpoint` context, not `backend`)

### Variables for requests
You can use the following variables inside the `check_expr`:

- `req_method`: Returns the method of this endpoint, e.g.: `GET`
- `req_path`: The path used to access this endpoint, e.g: : `/foo`
- `req_params`: An object with all the placeholder `{parameters}` declared in the endpoint . All **parameters capitalize the first letter**. E.g.: An `"endpoint": "/v1/users/{id_user}"` will set a variable `req_params.Id_user` containing the value of the parameter passed in the request. When you use the [sequential proxy](/docs/v2.11/endpoints/sequential-proxy/#chaining-the-requests) you also have under `req_params.RespX_field` the response of a previous backend call (where X is the sequence number and `field` the object you want to retrieve.
- `req_headers`: An array with all the headers received. The value of the array is at the same time another array, as you can have a header declared multiple times (e.g., multiple cookies with `Set-Cookie`). You can access headers like this: `req_headers['X-Forwarded-For']`. Notice that no matter how the header is written, you must access it using the **canonical form** (a header `x-SOME-thing` must be accessed as `X-Some-Thing`).
- `req_querystring`: An Object with all the query strings that the user passed to the endpoint (not anything you wrote on the backend `url_pattern`). Remember that no query strings pass unless they are in the `input_query_strings` list. Notice that querystrings, unlike `req_params`, are NOT capitalized. The `req_querystring.foo` will also return an array as a query string can contain multiple values (e.g: `?foo=1&foo=2`).
- `now`: An object containing the current timestamp, e.g:
  `timestamp(now).getDayOfWeek()`

### Variables for responses
You can use the following variables inside the `check_expr`:

- `resp_completed`: Boolean whether all the data has been successfully
  retrieved
- `resp_metadata_status`: Returns an integer with the StatusCode
- `resp_metadata_headers`: Returns an array with all the headers of the response
- `resp_data`: An object with all the data captured in the response. Using the dot notation, you can access its fields, e.g.:`resp_data.user_id`. If you use the `group` operator in the backend, then you need to add it to access the object, e.g., `resp_data.mygroup.user_id`
- `now`: An object containing the current timestamp

{{< note title="A note on response metadata" >}}
The response metadata is only filled for no-op pipes. In non no-op cases it will be always empty, and the pipe will end the execution by itself if the status code is not 200/201.
{{< /note >}}

### Variables for the JWT rejecter
You can also use CEL expressions during the JWT token validation. It only works if you have a configured `auth/validator` and when the CEL expression is at the `endpoint` level. Use the `JWT` variable to access its metadata.

Here's an example of expression you could use in an `endpoint`:
```js
has(JWT.user_id) && has(JWT.enabled_days) && (timestamp(now).getDayOfWeek() in JWT.enabled_days)
```
This example checks that the JWT token contains the metadata `user_id` and
`enabled_days` with the macro `has()`, and then checks that today's weekday is within one of the allowed days to see the endpoint.

And the required configuration would be:

```json
{
    "endpoint": "/nick/{nick}",
    "extra_config": {
        "validation/cel": [
            {
                "check_expr": "has(JWT.user_id) && has(JWT.enabled_days) && (timestamp(now).getDayOfWeek() in JWT.enabled_days)"
            }
        ],
        "auth/validator": {
            "alg": "RS256",
            "jwk_url": "https://example.com/.well-known/jwks.json",
            "cache": true
        }
    }
}
```

Notice that the `JWT` variable is **unset** when you evaluate expressions in the `backend`. If you want to check JWT claims in a `backend` context, you must [propagate their values as headers](/docs/v2.11/authorization/jwt-validation/#propagate-jwt-claims-as-request-headers), and then work with headers.

This is an example of accessing claims in a `backend` expression, through propagated claims:

```json
{
  "$schema": "https://www.krakend.io/schema/v2.11/krakend.json",
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/example",
      "input_headers": ["X-User-Id"],
      "extra_config": {
        "auth/validator": {
          "alg": "RS256",
          "jwk_url": "https://example.com/.well-known/jwks.json",
          "cache": true,
          "propagate_claims": [
            [
              "user_id","X-User-Id"
            ]
          ]
        }
      },
      "backend": [
        {
          "url_pattern": "/example",
          "host":["https://example.com"],
          "extra_config": {
            "validation/cel": [
              {
                "check_expr": "size(req_headers['X-User-Id']) == 1"
              }
            ]
          }
        }
      ]
    }
  ]
}
```

Notice that because we are in the `backend` context, we do not access `JWT` in the expression, but to the propagated header. The target header declared under `propagate_claims` must be also declared under `input_headers` to work properly.

## CEL Syntax and examples
See the CEL [language definition](https://github.com/google/cel-spec/blob/master/doc/langdef.md) for the complete list of supported options.

The following example snippets demonstrate how to check requests and responses.

### Example: Discard an invalid request before reaching the backend
The following example demonstrates how to reject a user request that does not fulfill a specific expression, checking at the endpoint level that when `/nick/{nick}` is called, a constraining format applies. More specifically, the example requires that the parameter `{nick}` matches the expression `k.*`:

```json
{
    "endpoints": [
        {
            "endpoint": "/nick/{nick}",
            "extra_config": {
                "validation/cel": [
                    {
                        "check_expr": "req_params.Nick.matches('k.*')"
                    }
                ]
            }
        }
    ]
}
```

With this configuration, any request to `/nick/kate` or `/nick/kevin` will make it to the backend, while a request to `/nick/ray` will be immediately rejected (`backend` section omitted intentionally for simplification purposes)

### Example: Check if the backend response has a specific field or abort
This example can be copied/pasted into a new configuration. The CEL validation happens at the backend level. After querying the backend, the CEL expression checks that a field `company` exists inside the response body. If the user does not have that field, the call to the endpoint will fail:

```json
{
    "version": 3,
    "endpoints": [
        {
            "endpoint": "/nick/{nick}",
            "backend": [
                {
                    "host": ["https://api.github.com"],
                    "url_pattern": "/users/{nick}",
                    "allow": ["name","company"],
                    "group": "github",
                    "extra_config": {
                        "validation/cel": [
                            {
                                "check_expr": "'company' in resp_data.github"
                            }
                        ]
                    }
                }
            ]
        }
    ]
}
```

Also, notice how we are accessing a `github` element in the data, a new attribute added by KrakenD thanks to the `group` functionality (it does not exist in the origin API). The takeaway is that the CEL evaluation is applied **after** KrakenD has processed the backend.

### Example: Match a query string parameter with multiple values
This example validates that an array query string parameter contains a given value. Many APIs describe array query string parameters with a `[]` suffix to denote that it's an array to the backend service. CEL syntax can reference these types of parameters.

In this case, an API operation accepts an array query string parameter named `foo`. The backend service's platform requires this passed to the API as `?foo[]=bar&foo[]=baz` in the query string.

KrakenD intercepts the parameter as the suffixed `foo[]`, so that's what must be allowed in your `input_query_strings` list. Since the parameter name contains the `[]` characters then it must be referred to as a map key instead of dot-notation in CEL syntax.

The following config uses CEL validation to block requests that do not have `foo[]` defined in the query string or do not have "bar" in the `foo[]` array:

```json
{
    "endpoint": "/example",
    "input_query_strings": [
        "foo[]"
    ],
    "backend": [
        {
            "host": ["api.example.com"],
            "url_pattern": "/example",
            "extra_config": {
                "validation/cel": [
                    {
                        "check_expr": "has(req_querystring['foo[]']) && 'bar' in req_querystring['foo[]']"
                    }
                ]
            }
        }
    ]
}
```

Note: this snippet applies CEL validation to a single backend. Apply CEL validation to an endpoint to validate across all backends.

If your application does not require the `[]` suffix in these parameters (clients pass in `?foo=bar&foo=baz` instead of `?foo[]=bar& foo[]=baz`) then omit the `[]` suffix from the parameter name in your `input_query_strings` list and refer to the parameter as `req_querystring.foo` in your `validation/cel` config.

### Example: Time-based access
Let's close the access to the API endpoint during the weekend:

```json
{
    "endpoint": "/weekdays",
    "extra_config": {
        "validation/cel": [
            {
                "check_expr": "(timestamp(now).getDayOfWeek() + 6) % 7 <= 4"
            }
        ]
    }
}
```
Note: The function `getDayOfWeek()` starts at `0` (Sunday), so the only days with a `mod <=4 ` are 0 and 6.

### Example: Use custom data from JWT payload
Let's say that the JWT token the user sent contains an attribute named `enabled_days` in its payload. This attribute lists all the integers representing which days the resource can be accessed:

```json
{
    "endpoint": "/combination/{id}",
    "extra_config": {
        "validation/cel": [
            {
                "check_expr": "has(JWT.user_id) && has(JWT.enabled_days) && (timestamp(now).getDayOfWeek() in JWT.enabled_days)"
            }
        ]
    }
}
```
The expression checks that the JWT token has both the `user_id` and the `enabled_days` and that today is good.

### Example: Conditional call of sequential backends (a.k.a "skip backends")
The following example is a bit more complex, as it **combines the sequential proxy with the CEL component**. You can copy and paste this example and start KrakenD with the `krakend run -d` flag.

```json
{
    "version": 3,
    "debug_endpoint": true,
    "host": [
        "http://localhost:8080"
    ],
    "endpoints": [
        {
            "endpoint": "/cel",
            "input_query_strings": [
                "foo"
            ],
            "backend": [
                {
                    "url_pattern": "/__debug/0"
                },
                {
                    "url_pattern": "/__debug/1?ignore={resp0_message}",
                    "group": "sequence1",
                    "extra_config": {
                        "validation/cel": [
                            {
                                "check_expr": "has(req_params.Resp0_message)"
                            }
                        ]
                    }
                },
                {
                    "url_pattern": "/__debug/2",
                    "group": "sequence2",
                    "extra_config": {
                        "validation/cel": [
                            {
                                "check_expr": "resp_data.sequence2.message == 'pong'"
                            }
                        ]
                    }
                },
                {
                    "url_pattern": "/__debug/3",
                    "group": "sequence3",
                    "extra_config": {
                        "validation/cel": [
                            {
                                "check_expr": "has(req_querystring.foo)"
                            }
                        ]
                    }
                },
                {
                    "url_pattern": "/__debug/4",
                    "group": "sequence4",
                    "extra_config": {
                        "validation/cel": [
                            {
                                "check_expr": "has(req_params.NEVER_CALLED_BACKEND)"
                            }
                        ]
                    }
                }
            ],
            "extra_config": {
                "proxy": {
                    "sequential": true
                }
            }
        }
    ]
}
```

Here is what it does:

- The backend 0 (first item in the `backend` list) calls the URL `/__debug/0`. It returns the object `{"message": "pong"}` as per the [debug endpoint](/docs/v2.11/endpoints/debug-endpoint/) definition.
- KrakenD will execute the rest of the backends one by one in the order defined, as the proxy is sequential.
- The next backend 1 will call `/__debug/1?ignore=pong`, as `pong` is the value of `resp0_message`. We are using an `ignore` querystring as if you were unable to modify your backend URL, but it could be part of the URL (e.g: `/__debug/1/{resp0_message}`). You must use at least one `resp_` variable to make KrakenD initialize them properly. In addition, as it has a CEL expression inside, this backend will be called **ONLY** if the backend 0 contains a `message` field. Notice that the backend does not have access to the body of the previous call, but it has access to the parameters in the `url_pattern`. Thus, we can use the `req_params` and access any `{parameter}` as `req_params.Resp0_parameter` (all parameters capitalize the first letter: **R**esp0)
- The backend 2 will always be triggered but will return the content only when the backend response has a `pong` string in the response. Notice that since we are working with a `group`ed response, the `sequence2` is inside the expression.
- The backend 3 will be called only if the original request contains a querystring ` foo'
- The backend 4 will never be called, as the endpoint does not define a `{NEVER_CALLED_BACKEND}` parameter

The expected response will be incomplete (as 1 or more backends will fail) and looks like:
{{< terminal title="Response" >}}
curl -iG http://localhost:8080/cel\?foo\=A
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
X-Krakend: Version 2.11
X-Krakend-Completed: false
Date: Tue, 22 Feb 2022 17:26:12 GMT
Content-Length: 114

{"message":"pong","sequence-1":{"message":"pong"},"sequence-2":{"message":"pong"},"sequence-3":{"message":"pong"}}
{{< /terminal >}}
