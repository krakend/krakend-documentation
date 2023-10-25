---
lastmod: 2022-01-18
old_version: true
date: 2022-01-18
linktitle: HTTP Transport settings
title: Advanced HTTP Transport settings
weight: 60
notoc: true
menu:
  community_v2.4:
    parent: "030 Service Settings"
---
When KrakenD communicates using http, it implements a concurrent-safe round tripper that supports HTTP, HTTPS, and HTTP proxies, and it caches connections for future re-use. This may leave many open connections when accessing many hosts. You can change the behavior of the transport layer using several settings presented below.

If you want to customize any of the settings below, they must be written at the top level of the configuration.

{{< schema version="v2.4" data="krakend.json" filter="disable_rest,dialer_timeout,dialer_keep_alive,dialer_fallback_delay,disable_compression,disable_keep_alives,max_idle_connections,max_idle_connections_per_host,idle_connection_timeout,response_header_timeout,expect_continue_timeout,client_tls">}}

Finally, the **TLS Handshake Timeout** is hardcoded to 10 seconds and cannot be changed.


## Override settings using environment vars
When you declare in the configuration file any of the HTTP server or transport settings declared above, you can [override its value through environment variables](/docs/v2.4/configuration/environment-vars/) when starting the server.

All the environment variables have the same name as the settings above in uppercase and with the `KRAKEND_` prefix. The following env vars are available:

- `KRAKEND_DIALER_TIMEOUT`
- `KRAKEND_DIALER_KEEP_ALIVE`
- `KRAKEND_DIALER_FALLBACK_DELAY`
- `KRAKEND_DISABLE_COMPRESSION`
- `KRAKEND_DISABLE_KEEP_ALIVES`
- `KRAKEND_MAX_IDLE_CONNECTIONS`
- `KRAKEND_MAX_IDLE_CONNECTIONS_PER_HOST`
- `KRAKEND_IDLE_CONNECTION_TIMEOUT`
- `KRAKEND_RESPONSE_HEADER_TIMEOUT`
- `KRAKEND_EXPECT_CONTINUE_TIMEOUT`


You can start KrakenD with the desired variables to override what you have in the configuration:

{{< terminal title="Term" >}}
KRAKEND_MAX_IDLE_CONNECTIONS_PER_HOST=200 krakend run -c krakend.json
{{< /terminal >}}

## Max IDLE connections
Having a high number of IDLE connections to every backend affects directly to the performance of the proxy layer. This is why you can control the number using the `max_idle_connections` setting. For instance:

```json
{
	"version": 3,
	"max_idle_connections": 150
}
```


KrakenD will close connections sitting idle in a "keep-alive" state when `max_idle_connections` is reached. If no value is set in the configuration file, KrakenD will use `250` by default.

Every ecosystem needs its own setting, have this in mind:

- If you set a number very high for `max_idle_connections` you might exhaust your system's port limit.
- If you set a number very low, new connections will be frequently created and a low rate of connection reuse will take place.
