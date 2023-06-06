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
meta:
  source: https://github.com/luraproject/lura
  scope:
  - service
  log_prefix:
  - "[SERVICE: HTTP Server]"
---
KrakenD starts an HTTP server to offer the API Gateway server. You can personalize some of the settings used to start the service and also override the default settings of the underlying Go [standard library](https://pkg.go.dev/net/http#Server).

If you want to customize any of the settings below, they must be written at the top level of the configuration.

{{< schema data="krakend.json" filter="port,cache_ttl,sequential_start,read_timeout,read_header_timeout,write_timeout,idle_timeout">}}

## Override settings using environment vars
When you declare in the configuration file any of the HTTP server settings declared above, you can [override its value through environment variables](/docs/configuration/environment-vars/) when starting the server.

All the environment variables have the same name are the same settings above in uppercase and with the `KRAKEND_` preffix. For instance, looking at the list of settings above, you could override:

- `KRAKEND_PORT`
- `KRAKEND_READ_TIMEOUT`
- `KRAKEND_READ_HEADER_TIMEOUT`
- `KRAKEND_WRITE_TIMEOUT`
- `KRAKEND_IDLE_TIMEOUT`
- etc...

You can start KrakenD with the desired variables to override what you have in the configuration:

{{< terminal title="Term" >}}
KRAKEND_PORT=8000 KRAKEND_READ_TIMEOUT="1s" krakend run -c krakend.json
{{< /terminal >}}
