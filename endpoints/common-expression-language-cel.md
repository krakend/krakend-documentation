---
lastmod: 2019-01-24
date: 2019-01-24
linktitle: Checking requests and responses
title: Checking requests and responses with the Common Expression Language (CEL)
weight: 80
since: 0.8
menu:
  documentation:
    parent: endpoints
---
There are times when you might want to include **additional logic** in the gateway
to decide if a request has to be fulfilled or not.

The [Common Expression Language (CEL)](https://github.com/devopsfaith/krakend-cel)
middleware enables Google's [CEL spec](https://github.com/google/cel-spec)
which implements common semantics for expression evaluation, and is a very
simple and powerful option to have full control during requests and responses.

When the CEL component is enabled, any amount of expressions to check both requests and responses can be set.

Then, during runtime, when an expression returns `false`, KrakenD does not return the content as the condition has failed. Otherwise, if all expressions returned `true` the content is served.

The CEL expressions have similar syntax to expressions in C/C++/Java/JavaScript and evaluate to a boolean condition. For instance:

    '::1' in req_headers['X-Forwarded-For']

This expression checks that the request header `X-Forwarded-For` contains the string `::1` (request comes from localhost).

CEL expressions can be used both during the **request** or the **response** of
the **backends** and the **endpoints**. The flow is:

- Request endpoint evaluation
- Request backend evaluation (N times)
- Response backend evaluation (N times)
- Response endpoint evaluation (can evaluate all merged data)

# Adding logic in the request
If you want to add some logic to decide whether or not to continue serving the request
to an endpoint or proxy to the next backend not, use a `req_*` variable.

## CEL variables
The following data is injected to the CEL evaluator for its inspection:

### Requests
- `req_method`: What is the method of this endpoint, e.g.: `GET`
- `req_path`: The path used to access this endpoing, e.g: : `/foo`
- `req_params`: Object with all the parameters sent with the request, e.g:
  `req_params.foo.var`
- `req_headers`: Array with all the headers received, e.g: `req_headers['X-Forwarded-For']`
- `now`: An object containing the current timestamp, e.g:
  `timestamp(now).getDayOfWeek()`

### Responses
- `resp_completed`: Boolean whether all the data has been successfully
  retrieved
- `resp_metadata_status`: Returns an integer with the StatusCode
- `resp_metadata_header`: Returns an array with all the headers of the response
- `resp_data`: An object with all the data captured in the response
- `now`: An object containing the current timestamp

### JWT rejecter
CEL expressions can also be used during the JWT token validation. Use the `JWT`
variable to access its metadata. For instance:


    has(JWT.user_id) && has(JWT.enabled_days) && (timestamp(now).getDayOfWeek() in JWT.enabled_days)

This example checks that the JWT token contains the metadata `user_id` and
`enabled_days` with the macro `has()`, and then checks that today's weekday  is
within one of the allowed days to see the endpoint.

# CEL Syntax
See the [language definition](https://github.com/google/cel-spec/blob/master/doc/langdef.md)
