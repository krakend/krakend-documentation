---
lastmod: 2021-05-02
date: 2019-10-07
linktitle:  Healthcheck
title: The health endpoint
weight: 10000
menu:
  community_current:
    parent: "040 Endpoint Configuration "
notoc: true
---

If you place a balancer in front of KrakenD, such as an ELB, you can check KrakenD health using a **TCP port check**. If, on the other hand, you need an **HTTP endpoint** in systems like Kubernetes, use the internal endpoint `/__health`.

## The `/__health` endpoint

The health endpoint, or the **ping endpoint**, works without any specific configuration as KrakenD automatically adds it.

For instance, see the **simplest possible `krakend.json`**:

{{< terminal title="Simplest configuration file" >}}
cat krakend.json
{ "version: 2 }
{{< /terminal >}}

Start this server and query the `/__health` endpoint. You'll have a `200` response code from KrakenD with the following content:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/__health
{"status":"OK"}
{{< /terminal >}}


## Custom alternatives to the health endpoint
If, for any reason, you don't want to use the `/__health` endpoint and its content and want to have your custom response, the simplest solution is to use  [stub data](/docs/endpoints/static-proxy/) to alter the response.

{{< note title="Avoid adding dependencies in your health check" >}}
When setting custom health checks, try not to use external backends connected to databases to determine if KrakenD has to be reloaded or not.
{{< /note >}}

A custom health configuration could look like this:

{{< highlight json >}}
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
                        "custom": "response",
                        "foo": "bar"
                    },
                    "strategy": "always"
                    }
                }
            },
            "backend": [
            {
                "url_pattern": "/__health",
                "host": [
                    "http://localhost:8080"
                ]
            }
            ]
        }
        ]
    }
{{< /highlight >}}

In this configuration, KrakenD connects to itself, but instead of returning the content of the internal health endpoint, it sets the `data` defined in the static structure. Notice that the listening `port` in the configuration and the `host` match your deployed KrakenD.

The response content of this custom `/health` endpoint is:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/health
{"custom": "response","foo": "bar"}
{{< /terminal >}}
