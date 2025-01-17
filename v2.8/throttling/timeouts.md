---
lastmod: 2022-10-11
old_version: true
date: 2018-01-04
linktitle: Timeouts
title: API Throttling and Timeout Management
description: Learn how to effectively manage API throttling and timeouts with KrakenD API Gateway to ensure optimal performance and prevent abuse
weight: 200
menu:
  community_v2.8:
    parent: "040 Routing and Forwarding"
---

Being KrakenD, an API Gateway that talks to other services, controlling the waiting times for different aspects is crucial. KrakenD will allow you to fine-tune these settings.

The timeouts can apply to:

- **The duration of the whole pipe** (from user request to user response)
- **The HTTP request related timeouts**

Additionally, you can control the number of [**maximum IDLE connections**](/docs/v2.8/service-settings/http-transport-settings/).

## Default timeout
The `timeout` key in the `krakend.json` at the root level is used to apply a **default timeout** for those endpoints that do not specify one. The duration you write in the timeout represents the **whole duration of the pipe**, so it counts the time all your backends take to respond and the processing of all the components involved in the endpoint (the request, fetching data, manipulation, etc.). You can see it is an **end-user timeout**, the maximum amount of time a user will wait for a response.

```json
{
    "version": 3,
    "timeout": "2000ms"
}
```
## Endpoint timeout
Probably the most convenient way to work with timeouts is by having a default `timeout` value in the root level with reasonable values. The default timeout will apply to all endpoints not setting a value and then set a `timeout` to those endpoints having a very different nature (like uploading files or heavy processing).

To do so, place it inside the desired endpoint:

```json
{
    "version": 3,
    "timeout": "2000ms",
    "endpoints": [
        {
            "endpoint": "/default-timeout"
        },
        {
            "endpoint": "/different-timeout",
            "timeout": "1s"
        }
    ]
}
```

The example above will use **1 second** timeout for the `/different-timeout` endpoint and **2 seconds** (expressed in milliseconds) for any the other endpoint. Note that the `backend` section is omitted for better reading.

## Context deadline exceeded
What happens when the timeout is reached? KrakenD will exhaust the specified `timeout` trying to fetch and process the content, but if that happens, the user will receive an HTTP status code `500 Internal Server Error` when there is no content to return.

If you connected to more than one backend and you have at least a valid response, the gateway will return a `200` status code with a **partial response** (the backends that worked)

The logs will show `context deadline exceeded` every time that you hit a timeout.

## Golden rule for setting timeouts
When setting timeouts, you always have to respect the following rule:

```
Client timeout > KrakenD timeout > Your API timeout
```

From the consumer to the provider, the timeouts need to decrease. For instance, your javascript application should have a timeout greater than KrakenD timeouts, and KrakenD should have a greater timeout than your backend API.

## HTTP Server and Transport timeouts
In addition to the timeouts, you can configure the specific HTTP transport timeouts and the HTTP server timeout.

For more information, see:

- [HTTP Server settings](/docs/v2.8/service-settings/http-server-settings/)
- [HTTP Transport settings](/docs/v2.8/service-settings/http-transport-settings/)

All the settings in these sections work just like the pipeline timeouts. However, if you place the values in an endpoint, they will override the default setting.
