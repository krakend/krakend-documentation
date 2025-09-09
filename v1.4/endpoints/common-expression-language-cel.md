---
lastmod: 2020-03-29
old_version: true
date: 2019-01-24
linktitle: Checking requests and responses
title: Checking requests and responses with the Common Expression Language (CEL)
weight: 80
images: ["/images/documentation/krakend-cel.png"]
menu:
  community_v1.4:
    parent: "040 Endpoint Configuration"
meta:
  since: v0.8
  source: https://github.com/krakend/krakend-cel
  namespace:
  - github.com/devopsfaith/krakend-cel
  scope:
  - endpoint
  - backend
---
There are times when you might want to include **additional logic** in the gateway to decide if a request has to be fulfilled or not.

The [Common Expression Language (CEL)](https://github.com/krakend/krakend-cel)
middleware enables Google's [CEL spec](https://github.com/google/cel-spec)
which implements common semantics for expression evaluation, and is a very
simple and powerful option to have full control during requests and responses.

When the CEL component is enabled, any amount of expressions to check both requests and responses can be set.

Then, during runtime, when an expression returns `false`, KrakenD does not return the content as the condition has failed. Otherwise, if all expressions returned `true`, the content is served.

The CEL expressions have a similar syntax to expressions in C/C++/Java/JavaScript and evaluate to a boolean condition. For instance:

    '::1' in req_headers['X-Forwarded-For']

This expression checks that the request header array `X-Forwarded-For` contains an item with a value matching `::1` (request comes from localhost).

CEL expressions can be used in five different places: during the **request** and the **response** of both **backends** and **endpoints** (see the blue dots in the image), and in the router layer as a JWT rejecter. The flow is:

![The 5 CEL places](/images/documentation/krakend-cel.png)

- **JWT** evaluation
- **Endpoint request** evaluation
- **Backend request** evaluation (per N backends)
- **Backend response** evaluation (per N backends)
- **Endpoint response** evaluation (can evaluate all merged data)

## Configuration
The CEL component goes inside the `extra_config` of your `endpoints` or your `backend` using the namespace `github.com/devopsfaith/krakend-cel`.

Depending on the place where you put the `extra_config`, the expressions will be checked at the `endpoint` level, or the `backend` level.

For instance, you might want to reject users that do not adhere to some criteria related to the content in their JWT token. There is no reason to delay this check, and you would place the check at the endpoint level, right before hitting any backend. In another scenario, you might want to make sure that the response of a specific backend contains a must-have field; that configuration would go under the `backend` section, and isolated from the rest of sibling backends under the same endpoint umbrella.

The configuration is as follows:

    "extra_config":{
      "github.com/devopsfaith/krakend-cel": [
        {
          "check_expr": "CONDITION1 && CONDITION2"
        }
      ]
    }

- `check_expr`: The expression that evaluates as a boolean, you can write any conditional. If all stacked conditions are *true* the request continues, *false*, it fails to retrieve data from the token, the request, or the response. The expressions can use a set of **variables**, shown in the sections below.


{{< note title="A note on client headers" >}}
When **client headers** are needed, remember to add them under [`headers_to_pass`](/docs/v1.4/endpoints/parameter-forwarding/#headers-forwarding) as KrakenD does not forward headers to the backends unless declared in the list.
{{< /note >}}


## Adding logic in the requests and responses.
There are three different ways to access the metadata of requests and responses to decide whether or not to continue serving the user command.

- Use a `req_` variable to access **request** data.
- Use a `resp_` variable to access **response** data.
- Use the `JWT` variable to access the **payload of the JWT**.

See the data that is injected to the CEL evaluator for its inspection below.

### Variables for requests
The following variables can be used inside the `check_expr`:

- `req_method`: What is the method of this endpoint, e.g.: `GET`
- `req_path`: The path used to access this endpoint, e.g: : `/foo`
- `req_params`: Object with all the parameters sent with the request, e.g:
  `req_params.Foo.var`. Notice that **parameters capitalize the first letter**
- `req_headers`: Array with all the headers received, e.g: `req_headers['X-Forwarded-For']`
- `now`: An object containing the current timestamp, e.g:
  `timestamp(now).getDayOfWeek()`

### Variables for responses
The following variables can be used inside the `check_expr`:

- `resp_completed`: Boolean whether all the data has been successfully
  retrieved
- `resp_metadata_status`: Returns an integer with the StatusCode
- `resp_metadata_headers`: Returns an array with all the headers of the response
- `resp_data`: An object with all the data captured in the response
- `now`: An object containing the current timestamp

### Variables for the JWT rejecter
CEL expressions can also be used during the JWT token validation. Use the `JWT`
variable to access its metadata. For instance:


    has(JWT.user_id) && has(JWT.enabled_days) && (timestamp(now).getDayOfWeek() in JWT.enabled_days)

This example checks that the JWT token contains the metadata `user_id` and
`enabled_days` with the macro `has()`, and then checks that today's weekday  is
within one of the allowed days to see the endpoint.

## CEL Syntax and examples
See the CEL [language definition](https://github.com/google/cel-spec/blob/master/doc/langdef.md) for the full list of supported options.

The following example snippets demonstrate how to check requests and responses.

### Example: Discard an invalid request before reaching the backend
The following example demonstrates how to reject a user request that does not fulfill a specific expression, checking at the endpoint level that when `/nick/{nick}` is called, a constraining format applies. More specifically, the example requires that the parameter `{nick}` matches the expression `k.*`:

    "endpoints": [
        {
          "endpoint": "/nick/{nick}",
          "extra_config":{
            "github.com/devopsfaith/krakend-cel": [
              {
                "check_expr": "req_params.Nick.matches('k.*')"
              }
            ]
          },
          "backend": [{...}]
        }

With this configuration, any request to `/nick/kate` or `/nick/kevin` will make it to the backend, while a request to `/nick/ray` will be immediately rejected.

### Example: Check if the backend response has a specific field or abort
This example can be copy/pasted into your configuration as it connects to an existing external API. The CEL validation happens at the backend level. After the backend has been queried, the CEL expression checks that a field `website` exists inside the response body. If the user does not have the field, the call to the endpoint will fail:

    "endpoints": [
    {
      "endpoint": "/nick/{nick}",
      "backend": [
        {
          "host": [
            "https://api.bitbucket.org"
          ],
          "url_pattern": "/2.0/users/{nick}",
          "allow": [
            "display_name",
            "website"
          ],
          "group": "bitbucket",
          "extra_config":{
            "github.com/devopsfaith/krakend-cel": [
              {
                "check_expr": "'website' in resp_data.bitbucket"
              }
            ]
          }
        }
      ]
    }

Also, notice how we are accessing a `bitbucket` element in the data, which is a new attribute added by KrakenD thanks to the `group` functionality (it does not exist in the origin API). The point here is that the CEL evaluation is applied **after** KrakenD has processed the backend.

### Example: Time-based access
Because we engineers prefer not to paged over the weekends when backends go down, let's close the access to them during the weekend, so they are 100% operational :)

    {
      "endpoint": "/weekdays",
      "extra_config":{
        "github.com/devopsfaith/krakend-cel": [
          {
            "check_expr": "(timestamp(now).getDayOfWeek() + 6) % 7 <= 4"
          }
        ]
    }

Note: The function `getDayOfWeek()` starts at `0` (Sunday), so the only days with a `mod <=4 ` are 0 and 6.

### Example: Use custom data from JWT payload
Let's say that the JWT token the user sent contains an attribute named `enabled_days` in its payload. This attribute lists all the integers representing which days the resource can be accessed:

    {
      "endpoint": "/combination/{id}",
      "extra_config":{
        "github.com/devopsfaith/krakend-cel": [
          {
            "check_expr": "has(JWT.user_id) && has(JWT.enabled_days) && (timestamp(now).getDayOfWeek() in JWT.enabled_days)"
          }
        ]
    }

The expression checks that the JWT token has both the `user_id` and the `enabled_days` and that today is a valid day.