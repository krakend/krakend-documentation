---
lastmod: 2020-10-05
date: 2019-10-07
linktitle:  Healthcheck
title: Adding a health check endpoint (ping)
weight: 10000
notoc: true
menu:
  documentation:
    parent: endpoints
---

If you place a balancer in front of KrakenD, such as an ELB, you can check KrakenD health using a **TCP port check**. If, on the other hand, you need an **HTTP endpoint** in systems like Kubernetes, use `/__health`.

## The `/__health` endpoint

The health endpoint in KrakenD works out of the box without adding any configuration in the server. Just point your checks to `/__health` and you'll have a `200` response code from KrakenD when the system is running, e.g.:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/__health
{"status":"OK"}
{{< /terminal >}}

## Alternative ways of doing a health check (prior to 1.2)

If for any reason you can't or don't want to use the `/__health` endpoint , **the result can be achieved using different strategies**.

Some of them can be:

- Add a `/health` endpoint in the configuration with [stub data](/docs/endpoints/static-proxy/)
- Use the `/__debug/` endpoint with your desired log level as the health check.
- Serve a static file through the [static Martian modifier](/docs/backends/martian/) as `/health`.
- Pass the `/health` to another backend (although you are checking the backend, not KrakenD)

See examples below.

### Simple `/health` endpoint
A very simple option to add a health option is to create a **KrakenD endpoint that connects to itself** as the backend, going through the router and proxy layers. The configuration would look like this:

{{< highlight json >}}
    {
        "version": 2,
        "port": 8080,
        "endpoints": [
        {
            "endpoint": "/health",
            "backend": [
            {
                "url_pattern": "/__debug/health",
                "host": [
                    "http://localhost:8080"
                ]
            }
            ]
        }
        ]
    }
{{< /highlight >}}

Notice that the listening `port` in the configuration and the `host` in the backend (8080) match.

{{< note title="When starting the server..." >}}
Make sure you pass the `-d` flag to start KrakenD to enable the [`/__debug` endpoint](/docs/endpoints/debug-endpoint/). It's **perfectly safe** in production, as it returns a *ping-pong* answer, and nothing else
{{< /note >}}

The response content of the `/health` endpoint is:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/health
{"ping":"pong"}
{{< /terminal >}}

You can customize this response with stub data.

### How to add a `/health` endpoint with a custom response
An additional setting to the configuration above is to add the [static proxy](/docs/endpoints/static-proxy/) functionality wich allows you to return stub data. With its strategy `always` we are going to make sure that we always return the `data` we have declared in the configuration.

When using this strategy, the backend can be a fake host (then you don't need to start KrakenD with `-d`), as KrakenD returns the static `data` every time.

 {{< highlight json "hl_lines=11" >}}
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
{{< /highlight >}}

The response content of the `/health` endpoint is:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/health
{"status":"OK"}
{{< /terminal >}}
