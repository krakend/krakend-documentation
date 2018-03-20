---
aliases:
- /faq
date: 2016-10-26
linktitle: KrakenD F.A.Q
title: Frequently Asked Questions
description: Frequently Asked Questions
---

# Understanding the behaviour
## I am seeing frequent `503` errors in the logs
E.g:

    2016/11/13 - 18:01:18 | 200 |    5.352143ms | ::1 |   GET     /frontpage
    2016/11/13 - 18:01:18 | 503 |       5.662µs | ::1 |   GET     /frontpage
    2016/11/13 - 18:01:18 | 503 |       5.662µs | ::1 |   GET     /frontpage

The `maxRate` setting defines the maximum number of requests allowed in a single second to an endpoint or backend. When this number is reached, subsequent connections are rejected with a `503` error. This limitation is optional and is usually set to avoid hammering your own backends and compromising their stability.

### Solution
Increase the `maxRate` number or disable it (`maxRate = 0`). This setting can be set globally for all the endpoints,
or overridden individually per endpoint.

Remember: failing fast is always better than overloading your infrastructure and degrading the quality of your entire services.

## I am having empty responses
The main reasons for having responses are:

- **Timeout** when connecting the backend. The KrakenD service will cut the connection and will return and empty response if the backend does not respond in the time you set through the `timeout` variable. This variable is usually written in magnitude of **milliseconds**.
- **Invalid JSON/XML**. When the backend received a malformed object as response and could not decode it.

See the solutions below.

### Solution to cuts by timeout
Whenever possible add caching layers in your backends, scale the infrastructure, etc. so that can answer requests in a
decent time. Increasing the `timeout` variable is an option but but should be always **your last option**. If your
backends are not able to respond in a short time think that when you increment the timeout what you really do is
to block connections waiting for the backend. The memory consumption will increase and the number of connections you can
open are finite. In a gateway your focus should be freeing the connections as soon as possible.

Values above `2000ms` are not recommended.

### Solution to invalid responses.
Make sure your backend sources return valid Json/Xml/... data. Try any on-line service to check the validity and format
of the returned content. If the response of your API is a collection, e.g: response comes inside brackets `[]`, then make sure to mark the option `Treat the response as a collection, not an object.` in the form.


## Reserved endpoints
The following names cannot be used as endpoint names as they are reserved:

    /__debug/
    /favicon.ico