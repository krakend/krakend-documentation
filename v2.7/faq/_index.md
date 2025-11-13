---
old_version: true
date: 2016-10-26
lastmod: 2023-02-06
linktitle: KrakenD F.A.Q
title: Frequently Asked Questions (FAQ) - KrakenD API Gateway
weight: -1
description: Get quick answers to frequently asked questions about how KrakenD works and the reason behind different responses.
menu:
  community_v2.7:
    parent: "999 Frequently Asked Questions"
---

## I am getting a `200` status when the backend returns a `201`
E.g:

    2017/01/19 - 10:31:27 | 200 |    1.134431ms | ::1 |   POST     /users

### Explanation

The gateway will default sending an HTTP status 200 if the backend returns a 200 or a 201. If you want to return the original `201` you must use [`no-op` encoding](/docs/v2.7/endpoints/no-op/).

## I am getting a `500` status when the backend returns anything but `200`, `201` or redirects
E.g:

    2017/01/19 - 10:31:37 | 500 |    1.007191ms | ::1 |   POST     /users_ko

### Explanation

The gateway will default send a 500 HTTP status code if the backend returns any status above 400. You can [change this strategy](/docs/v2.7/backends/detailed-errors/).

## I am seeing frequent `503` errors in the logs
E.g:

    2016/11/13 - 18:01:18 | 200 |    5.352143ms | ::1 |   GET     /frontpage
    2016/11/13 - 18:01:18 | 503 |       5.662µs | ::1 |   GET     /frontpage
    2016/11/13 - 18:01:18 | 503 |       5.662µs | ::1 |   GET     /frontpage

The `max_rate` setting defines the maximum number of requests allowed in a single second to an endpoint or backend. When this number is reached, subsequent connections are rejected with a `503` error. This limitation is optional and is usually set to avoid hammering your own backends and compromising their stability.

### Solution
Increase the `max_rate` number or disable it (`max_rate = 0`). This setting can be set globally for all the endpoints,
or overridden individually per endpoint.

Remember: failing fast is always better than overloading your infrastructure and degrading the quality of your entire services.

## I have empty responses
The main reasons for having responses are:

- **Timeout** when connecting the backend. The KrakenD service will cut the connection and will return an empty response if the backend does not respond in the time you set through the `timeout` variable. This variable is usually written in a magnitude of **milliseconds**.
- **Invalid JSON/XML**. When the backend received a malformed object as response and could not decode it.

See the solutions below.

### Solution to cuts by timeout
When there is a timeout, you'll see the `context deadline exceeded` in the log, which means only one thing: KrakenD couldn't get the info on time (because of a network problem or backend slowness).

    Error #01: context deadline exceeded

Whenever possible, add caching layers in your backends, scale the infrastructure, etc., so backends answer on time. **Increasing the `timeout` variable should always be your last option**. If your backends are not able to respond in a short time, think that when you increment the timeout, what you do is block connections waiting for the backend. Memory consumption will increase, and the number of connections you can open is finite. In a gateway, your focus should be freeing the connections as soon as possible.

Values above `2000ms` are not recommended.

There are other times when KrakenD cannot reach the host due to a networking issue.

### Solution to invalid responses.
Make sure your backend sources return valid JSON/XML/... data. Try any online service to check the validity and format
of the returned content. If the response of your API is a collection, e.g., response comes inside brackets `[]`, then make sure to mark the option `Treat the response as a collection, not an object.` in the form.

## I've upgraded to v2.x, and I start to see `context canceled` errors
You start to see in the log `context canceled` errors, but before the upgrade, you didn't.

### Solution
The error means that the client disconnected while consuming the content. KrakenD can only do something if a client cuts the connection in the middle of the transmission: notify it.

You are seeing these errors now because the latest versions of KrakenD refactored how the router logging worked. Before this refactor, errors like this were showing in the access log (which was an incorrect approach) instead of the application log, where they belong.

The effect is that you are seeing them for the first time, but the disconnections were always there. If you check the older access logs in the stdout (not the application log), you will still find them.


## Reserved endpoints
The following endpoints are reserved, and you cannot use them (unless you disable them or rename them):

- `/__debug/` (disabled by default)
- `/__echo/` (disabled by default)
- `/__stats/`  (disabled by default)
- `/__health/` (can be renamed or disabled)

## I have found a vulnerability
If you think you have found a security problem, please [report us the vulnerability](/security-policy/)