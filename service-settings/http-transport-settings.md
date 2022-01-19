---
lastmod: 2022-01-18
date: 2022-01-18
linktitle: HTTP Transport settings
title: Advanced HTTP Transport settings
weight: 60
notoc: true
aliases: ["/docs/throttling/max-idle-connections/"]
menu:
  community_current:
    parent: "030 Service Settings"
---
When KrakenD communicates using http, it implements a concurrent-safe round tripper that supports HTTP, HTTPS, and HTTP proxies, and it caches connections for future re-use. This may leave many open connections when accessing many hosts. You can change the behavior of the transport layer using several settings presented below.




If you want to customize any of the settings below, they must be written at the top level of the configuration.


| Setting parameter | Type | Description |
|-------------------|---------------|-------------|
| dialer_timeout | *duration* | The timeout of the dial function for creating connections |
| dialer_keep_alive | *duration* | The amount of time you want to keep alive the connection |
| dialer_fallback_delay | *duration* |  Specifies the length of time to wait before spawning a fallback connection |
| disable_compression | *boolean* | When true prevents requesting compression with an `"Accept-Encoding: gzip"` request header when the Request contains no existing Accept-Encoding value. If the Transport requests gzip on its own and gets a gzipped response, it's transparently decoded. However, if the user explicitly requested gzip it is not automatically uncompressed. |
| disable_keep_alives | *boolean* | When true it disables HTTP keep-alives and will only use the connection to the server for a single HTTP request.|
| max_idle_connections | *integer* | The maximum number of idle (keep-alive) connections across all hosts. Zero means no limit. |
| max_idle_connections_per_host | *integer* | If non-zero, controls the maximum idle (keep-alive) connections to keep per-host. If zero, the default (`2`) is used. |
| idle_connection_timeout | *duration* | The maximum amount of time an idle (keep-alive) connection will remain idle before closing itself. Zero means no limit. |
| response_header_timeout | *duration* | If non-zero, specifies the amount of time to wait for a server's response headers after fully writing the request (including its body, if any). This time does not include the time to read the response body.|
| expect_continue_timeout | *duration* | If non-zero, specifies the amount of time to wait for a server's first response headers after fully writing the request headers if the request has an `"Expect: 100-continue"` header. Zero means no timeout and causes the body to be sent immediately, without waiting for the server to approve. This time does not include the time to send the request header.|

Finally, the **TLS Handshake Timeout** is hardcoded to 10 seconds and cannot be changed.


## Override settings using environment vars
When you declare in the configuration file any of the HTTP server settings declared above, you can [override its value through environment variables](/docs/configuration/environment-vars/) when starting the server.

All the environment variables have the same name are the same settings above in uppercase and with the `KRAKEND_` preffix. The following env vars are available:


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

{{< highlight json >}}
{
	"version": 3,
	"max_idle_connections": 150
}
{{< /highlight >}}


KrakenD will close connections sitting idle in a "keep-alive" state when `max_idle_connections` is reached. If no value is set in the configuration file, KrakenD will use `250` by default.

Every ecosystem needs its own setting, have this in mind:

- If you set a number very high for `max_idle_connections` you might exhaust your system's port limit.
- If you set a number very low, new connections will be frequently created and a low rate of connection reuse will take place.
