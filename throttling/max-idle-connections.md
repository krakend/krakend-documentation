---
lastmod: 2018-09-27
date: 2018-01-04
linktitle: Max IDLE connections
title: Maximum IDLE connections
weight: 60
notoc: true
menu:
  community_current:
    parent: "070 Traffic Management"
---

Having a high number of IDLE connections to every backend affects directly to the performance of the proxy layer. This is why you can control the number using the `max_idle_connections` setting. For instance:

	{
	  "version": 2,
	  "max_idle_connections": 150,
	  ...
	}

KrakenD will close connections sitting idle in a "keep-alive" state when `max_idle_connections` is reached. If no value is set in the configuration file, KrakenD will use `250` by default.

Every ecosystem needs its own setting, have this in mind:

- If you set a number very high for `max_idle_connections` you might exhaust your system's port limit.
- If you set a number very low, new connections will be frequently created and a low rate of connection reuse will take place.


For more information on the transport layer, see [MaxIdleConnsPerHost](https://golang.org/pkg/net/http/#Transport)
