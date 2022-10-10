---
lastmod: 2018-09-27
date: 2018-01-04
linktitle: Request timeouts
title: Timeouts
weight: 40
menu:
  community_current:
    parent: "070 Traffic Management"
---

Being KrakenD an API Gateway that talks to other services, being able to control the waiting times for different aspects is crucial. KrakenD will allow you to fine-tune these settings.

The timeouts can apply to:

- **The duration of the whole pipe** (from user request to user response)
- **The HTTP request related timeouts**

Additionally, you can control the number of [**maximum IDLE connections**](/docs/service-settings/http-transport-settings/).

## Global timeout
The `timeout` key in the `krakend.json` at the root level is used in the first place to apply a **default timeout** for the **whole duration of the pipe** (and not only the connection to the backends). The timeout takes into account all the time involved between the request, the fetching of the data, manipulation and any other middleware. You can see it is an **end-user timeout**.

```json
{
	"version": 3,
	"timeout": "2000ms"
}
```


## Endpoint-specific timeout
Even the `timeout` value in the root level sets a default timeout for all the endpoints, you can override it later on any specific endpoints.

To do so, just place it inside the desired endpoint:

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

## What happens when the timeout is reached?
When the timeout for the whole pipe is reached, the request is canceled and the user receives an HTTP status code `500 Internal Server Error`.

The logs will show a line like:
```
Context deadline exceeded
```

## HTTP Server and Transport timeouts
In addition to the timeouts for the whole pipe, you can configure the specific HTTP transport timeouts and the HTTP server timeout.

For more information see:

- [HTTP Server settings](/docs/service-settings/http-server-settings/)
- [HTTP Transport settings](/docs/service-settings/http-transport-settings/)

All the settings in these sections work just like the pipeline timeouts. If you place the values in an endpoint, they will override the global setting.
