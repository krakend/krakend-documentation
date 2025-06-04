---
lastmod: 2023-10-24
date: 2022-06-07
linktitle:  Service Settings
title: Service Settings Configuration
description: Service settings are optional flags that allow you to change how KrakenD behaves globally for all endpoints across configuration.
weight: -1000
notoc: true
menu:
  community_current:
    parent: "030 Service Settings"
dark_header_image: true
images:
- /images/documentation/hero/service-settings.png
---
We call **service settings** (or the service layer) those parameters that allow you to change how KrakenD behaves **globally** (and not to a specific call). They determine how you start the HTTP server, enforce security parameters, or define behavioral options like which reporting activities occur, to name a few examples.

Examples of service settings are, the [listening port](/docs/service-settings/http-server-settings/), [disabling keep alives](/docs/service-settings/http-transport-settings/), enabling [metrics and traces](/docs/telemetry/), [listening https](/docs/service-settings/tls/), or enabling [CORS](/docs/service-settings/cors/) to name a few.

If you haven't done it yet, read [ Understanding the configuration file](/docs/configuration/structure/).

All service settings are written directly in the root of the configuration file or its corresponding `extra_config`. So, for instance, here there is a configuration file describing a service listening on port 8080 with extended logging enabled:

```json
{
    "version": 3,
    "port": 8080,
    "output_encoding": "json",
    "host": ["http://myapi"],
    "timeout": "3s",
    "cache_ttl": "60s",
    "listen_ip": "192.168.1.3",
    "endpoints": [],
    "extra_config": {
      "telemetry/logging": {
        "level": "WARNING",
        "syslog": true,
        "stdout": true
      }
    }
}
```

The service accepts **numerous configuration options** that you'll find explained through the rest of the documentation, but here is a preview of the most important ones:

{{< schema data="krakend.json" filter="version,extra_config,port,listen_ip,endpoints,output_encoding,timeout,host,cache_ttl">}}

Other service-level settings you can add:

- [HTTP server settings](/docs/service-settings/http-server-settings/)
- [HTTP transport settings](/docs/service-settings/http-transport-settings/)
- [Router settings](/docs/service-settings/router-options/)
- [SSL/TLS](/docs/service-settings/tls/)
