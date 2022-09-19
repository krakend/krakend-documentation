---
lastmod: 2022-02-16
date: 2022-01-18
linktitle: HTTP Server settings
title: Advanced HTTP Server settings
weight: 50
notoc: true
menu:
  community_current:
    parent: "030 Service Settings"
---
KrakenD starts an HTTP server to offer the API Gateway server. You can personalize some of the settings used to start the service and also override the default settings of the underlying Go [standard library](https://pkg.go.dev/net/http#Server).

If you want to customize any of the settings below, they must be written at the top level of the configuration.

| Setting parameter | Type | Description |
|-------------------|---------------|-------------|
| `port`  | *integer* | The TCP port where KrakenD listens to. Recommended value is in the range 1024-65535 to run as an unpriviliged user. If you use ports under 1024 (e.g.: 80, 443) you'll need to start the service as root. Defaults to `8080`. |
| `cache_ttl`  | *duration* | Sets a default `Cache-Control: public, max-age=%d` header to all endpoints where `%d` is the conversion to seconds of any duration you wrote, indicating for how long the client can cache the content of the request. You can override this value per endpoint. Notice that KrakenD does not cache the content with this parameter, but tells the client how to do it. Defaults to `0s` (no cache). **For KrakenD cache, see [backend caching](/docs/backends/caching/)** |
| `sequential_start`  | *boolean* | A sequential start registers async agents in order when you run the server, allowing you to read the starting logs in sequential order. The startup speed depends on the number of async agents to register and the number of CPUs. A non-sequential start (default) is much faster, but logs are not in order. Defaults to `false`. |
| `read_timeout`| *duration* | Is the maximum duration for reading the entire request, including the body. Because `read_timeout` does not let Handlers make per-request decisions on each request body's acceptable deadline or upload rate, most users will prefer to use `read_header_timeout`. It is valid to use them both.|
| `read_header_timeout` | *duration* | The amount of time allowed to read request headers. The connection's read deadline is reset after reading the headers and the Handler can decide what is considered too slow for the body. |
|`write_timeout`| *duration* | The maximum duration before timing out writes of the response. It is reset whenever a new request's header is read. Like `read_timeout`, it does not let Handlers make decisions on a per-request basis.|
| `idle_timeout` | *duration* | The maximum amount of time to wait for the next request when keep-alives are enabled. If `idle_timeout` is zero, the value of `read_timeout` is used. If both are zero, there is no timeout. |

The *duration* is positive integer value with time units. Valid time units are: `ns` (nanoseconds), `us` or `µs` (microseconds), `ms` (milliseconds), `s` (seconds), `m` (minutes), `h` (hours - don't!)

Examples: `2s` for 2 seconds or `1500ms` for 1500 milliseconds.

## Override settings using environment vars
When you declare in the configuration file any of the HTTP server settings declared above, you can [override its value through environment variables](/docs/configuration/environment-vars/) when starting the server.

All the environment variables have the same name are the same settings above in uppercase and with the `KRAKEND_` preffix. The following env vars are available:

- `KRAKEND_PORT`
- `KRAKEND_READ_TIMEOUT`
- `KRAKEND_READ_HEADER_TIMEOUT`
- `KRAKEND_WRITE_TIMEOUT`
- `KRAKEND_IDLE_TIMEOUT`

You can start KrakenD with the desired variables to override what you have in the configuration:

{{< terminal title="Term" >}}
KRAKEND_PORT=8000 KRAKEND_READ_TIMEOUT="1s" krakend run -c krakend.json
{{< /terminal >}}
