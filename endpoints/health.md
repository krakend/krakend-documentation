---
lastmod: 2019-10-07
date: 2019-10-07
linktitle:  Health endpoint
title: Adding a health endpoint
weight: 110
notoc: true
menu:
  documentation:
    parent: endpoints
---

If you place a balancer in front of KrakenD, such as an ELB, you can check KrakenD health using a TCP port check. If, on the other hand, you need to have an HTTP endpoint like `/health` or `/ping` in systems like Kubernetes, you can do it in different ways.

Although there is no default health check implementation, the result can be achieved using different strategies. For instance:

- Add a `/health` endpoint in the configuration with [stub data](/docs/endpoints/static-proxy/) (see example below)
- Enable the `/__debug/` endpoint with your desired log level and use it as the health check.
- Use a [static Martian modifier](/docs/backends/martian/) and serve any static file as `/health`.
- Pass the `/health` to another backend (although you are checking the backend, not KrakenD)

## How to add a `/health` endpoint with stub data
The easiest option is to use the [static proxy](/docs/endpoints/static-proxy/) functionality. With this technique, we are going to add an endpoint that uses an unreachable (and fake) backend. As the data can't be fetched from KrakenD, the strategy `always` is going to make sure we return the desired static data that we want.

    {
        "version": 2,
        "port": 8080,
        "endpoints": [
        {
            "endpoint": "/health",
            "extra_config": {
            "github.com/devopsfaith/krakend/proxy": {
                "static": {
                "data": {
                    "status": "OK"
                },
                "strategy": "always"
                }
            }
            },
            "backend": [
            {
                "url_pattern": "/",
                "host": [
                    "http://fake-backend"
                ]
            }
            ]
        }
        ]
    }

In the previous example we are returning a `{ "status": "OK" }` every time the `/health` endpoint is called:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/health
{"status":"OK"}
{{< /terminal >}}