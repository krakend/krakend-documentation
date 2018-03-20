---
lastmod: 2018-01-04
date: 2018-01-04
linktitle: Timeouts
title: KrakenD - Timeouts
weight: 40
menu:
  main:
    parent: Throttling and Limits
---

Being KrakenD an API Gateway that talk to other services, being able to control the waiting times for different aspects is crucial. KrakenD will allow you to fine-tune this settings.

The timeouts can apply to:

- **The duration of the whole pipe** (from user request to user response)
- **The HTTP request related timeouts**

Additionally you can control the number of **maximum IDLE connections**.

# Pipe timeouts

## Global timeout
The `timeout` key in the `krakend.json` at the root level is used in the first place to apply a **default timeout** for the **whole duration of the pipe** (and not only the connection to the backends). The timeout takes into account all the time involved between the request, the fetching of the data, manipulation and any other middleware. You can see it is an end-user timeout.

	{
	  "version": 2,
	   "timeout": "2000ms",
	   ...
	}

## Endpoint-specific timeout
Even the `timeout` value in the root level sets a default timeout for all the endpoints, you can override it later on any specific endpoints.

To do so, just place it inside the desired endpoint:

	{
	  "version": 2,
	  "timeout": "2000ms",
	  "endpoints": [
	    {
	      "endpoint": "/splash",
	      "method": "GET",
	      "timeout": "1s"
	      ...
	    }
	  ]
	}

The example above will use 1 second timeout for the `/splash` endpoint and 2000 milliseconds for all the other endpoints.

## What happens when the timeout is reached?
When the timeout for a pipe is reached, the request is canceled and the user receives an HTTP status code `500 Internal Server Error`

# HTTP Request timeouts
In addition to the timeouts for the whole pipe, you can configure the specific HTTP layer timeouts.
All the settings below work in the same way the pipeline timeouts, if you place this values in an endpoint, they will override the global setting.

## HTTP Read Timeout
Maximum duration for reading the entire HTTP request, including the body.

This timeout does not let Handlers make per-request decisions on each request body's acceptable deadline.

	{
		"version": 2,
		"read_timeout": "1s"
	}

## HTTP Write Timeout
Maximum duration before timing out writes of the response.

It is reset whenever a new request header is read. Like HTTP Read Timeout, it does not let Handlers make decisions on a per-request basis.

	{
		"version": 2,
		"write_timeout": "0s"  (no timeout)
	}


## HTTP Idle Timeout
Maximum amount of time to wait for the next request when keep-alives are enabled.

If IdleTimeout is zero, the value of ReadTimeout is used. If both are zero, ReadHeaderTimeout is used.

	{
		"version": 2,
		"idle_timeout": "0s"  (no timeout)
	}

## HTTP Read Header Timeout
Amount of time allowed to read request headers.

The connection's read deadline is reset after reading the headers and the Handler can decide what is considered too slow for the body.

	{
		"version": 2,
		"read_header_timeout": "10ms"
	}

# Time units
The time units you can use to specify timeouts are integers (not *floats*) using any of these units:

- Nanoseconds: `ns`
- Microseconds: `us`, or `µs`
- Milliseconds: `ms`
- Seconds: `s`

It is also available although you should never use them:

- Minutes: `m`
- Hours: `h`

Examples of equivalent timeouts in different units are:

	"timeout": "1s",
	"timeout": "1000ms",
	"timeout": "1000000us",
	"timeout": "1000000000ns",

