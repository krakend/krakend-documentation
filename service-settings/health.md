---
lastmod: 2022-01-19
date: 2019-10-07
linktitle:  Health check
title: API Health Check
weight: 1
aliases: ["/docs/endpoints/health/"]
menu:
  community_current:
    parent: "030 Service Settings"
notoc: true
---
The health endpoint (or the **ping endpoint**) allows you to query KrakenD to find out if it is ready to accept connections or not.

When KrakenD is up and running correctly, it exposes a `/__health` endpoint returning a `200` HTTP status code. It works automatically and without adding any specific configuration block.

## Health check response
When you query the `/__health` endpoint, you should expect a `200` response code or no response at all. There are no other status codes that you can receive from the health, as it reflects a binary answer: it's working, or it's not. So make sure to check for a `200` when you monitor the health of the service.

The content of the health endpoint provides extra information, but you don't need to parse its content to make automated decisions. For example, the content could look like this:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/__health
{
    "agents": {
        "some-agent": "2022-01-19 18:25:17.00225307 +0100 CET m=+0.031662879"
    },
    "now": "2022-01-19 18:38:38.084402465 +0100 CET m=+30.674738658",
    "status": "ok"
}
{{< /terminal >}}

There are three keys inside the response:

- `status` with an `ok` value simply tells you that the API is processing HTTP requests correctly. There is no other possible state if the server is up.
- `agents` is a map of all [Async Agents](/docs/async/) you have running on KrakenD. The map will be empty if you don't use them. When agents are running, the value shows the time of the last working ping.
- `now` is the current time in the server.

## Disabling or renaming the health endpoint
You might want to disable the `/__health` endpoint or rename it. Whatever is the reason, you can set these parameters in the [router options](/docs/service-settings/router-options/#customizing-the-health-endpoint).

## Custom alternatives to the health endpoint
If you'd like to have a health endpoint with a custom response, the simplest solution is to use [stub data](/docs/endpoints/static-proxy/) to alter the existing response.

{{< note title="Avoid adding dependencies in your health check" >}}
When setting custom health checks, try not to use external backends connected to databases to determine if KrakenD has to be reloaded or not.
{{< /note >}}

A custom health configuration could look like this:

{{< highlight json >}}
    {
        "version": 3,
        "port": 8080,
        "endpoints": [
        {
            "endpoint": "/health",
            "extra_config": {
                "proxy": {
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

The `/__health` endpoint will still be available, but you can rename it, or if your backend does not use it, disable it.