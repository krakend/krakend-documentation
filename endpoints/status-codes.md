---
lastmod: 2020-10-19
date: 2020-10-19
notoc: true
linktitle: Status Codes
title: Status Codes in KrakenD API Gateway
description: Learn how to interpret HTTP status codes in KrakenD API Gateway, ensuring accurate and meaningful responses to API consumers
weight: 25
menu:
  community_current:
    parent: "060 Request and Response Manipulation"
---

When consuming content through KrakenD, the status code returned to the client depends on the chosen configuration. Three different approaches impact status codes:

- Use KrakenD regular endpoints to get the status codes as designed by KrakenD
- Return the status code as provided by your backend server (see the [`no-op` encoding](/docs/endpoints/no-op/))
- Use custom logic to set specific status codes

## Default status codes of KrakenD endpoints

Unless the `no-op` encoding is set, the following status codes are the default behavior of any KrakenD endpoint.

| Status Code                 | When                               |
|-----------------------------|-------------------------------------------|
| `200 OK`                    | **At least** one backend returned a 200 or 201 status code on time. Completeness information provided by the `X-Krakend-Completed` header |
| `404 Not Found`             | The requested endpoint is not configured on KrakenD           |
| `400 Bad Request`           | Client made a malformed request, i.e. [json-schema](/docs/endpoints/json-schema/) validation failed         |
| `401 Unauthorized`          | Client sent an invalid JWT token or its claims |
| `403 Forbidden`             | The user is allowed to use the API, but not the resource, e.g.: Insufficient JWT [role](/docs/authorization/jwt-validation/), or [bot detector](/docs/throttling/botdetector/) banned it |
| `429 Too Many Requests`     | The client reached the rate limit for the endpoint |
| `503 Service Unavailable`   | All clients together reached the configured global rate limit for the endpoint |
| `500 Internal Server Error` | Default error code, and in general, when backends return any status above `400` |

### Why does KrakenD treat errors like a `500 Internal Server Error` by default?

In most cases, when there isn't a happy path, you'll see KrakenD returning a `500 Internal Server Error`. When KrakenD needs to combine in the final gateway response, there is no way to properly distinguish the status code from the backend and the one from the gateway itself. That's why all errors external to KrakenD are translated into a `500 Internal Server Error`.

To offer a gracefully degraded service when some backends fail, we leave the decision to the client on what to do by adding the header `X-Krakend-Completed: false` (some backends succeeded, others don't) and also by adding the [detailed errors](/docs/backends/detailed-errors/) feature.

## Returning the status codes of the backend

If what you need is returning the content of a backend service *as is*, then the [no-op encoding](/docs/endpoints/no-op/) will proxy the client call to the backend service without any manipulation. When the backend produces the response, it's passed back to the client, preserving its form: body, headers, status codes, and such.

An exception to this behavior is `30x` responses, which will be followed by the gateway even with `no-op` encoding. If your backend returns a `301` the client won't follow it, but the gateway.

## Returning other status codes

Default status codes can be overridden per endpoint, following different implementations.

- **[Using no-operation](/docs/endpoints/no-op/)**: When your call is not idempotent (i.e., a write operation), and you want the client to receive whatever the backend is responding.
- **[Using a Lua script](/docs/endpoints/lua/)**: To write in the configuration any logic, you need to evaluate and return a `custom_error`, with any status code of your choice.
- As `custom_error` will end the pipe execution. If you just want to alter the status code, you can (in a no-op pipe) use the `statusCode` dynamic helper on the response.
- **Injecting your own [HTTPStatusHandler](https://github.com/luraproject/lura/issues/102#issuecomment-373657911)**
