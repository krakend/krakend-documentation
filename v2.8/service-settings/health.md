---
lastmod: 2022-01-19
old_version: true
date: 2019-10-07
linktitle: Health check endpoint
title: API Health Check
weight: 1100
menu:
  community_v2.8:
    parent: "160 Monitoring, Logs, and Analytics"
---
The health endpoint (or the **ping endpoint**) allows you to query KrakenD to find out if it is ready to accept connections or not.

When KrakenD is up and running correctly, it exposes a `/__health` endpoint returning a `200` HTTP status code. It works automatically and without adding any specific configuration block. Nevertheless you can do customizations to it.

## Health check response
When you query the `/__health` endpoint, you should expect a `200` response code **or no response at all**. There are no other status codes that you can receive from the health, as it reflects a binary answer: it's working, or it's not. So make sure to check for a `200` when you monitor the health of the service.

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
- `agents` is a map of all [Async Agents](/docs/v2.8/async/) you have running on KrakenD. The map will be empty if you don't use them. When agents are running, the value shows the time of the last working ping.
- `now` is the current time in the server.

## Customizing the health endpoint
You might want to disable the `/__health` endpoint, rename it, or disable its access log. You can set several parameters as [`router` options](/docs/v2.8/service-settings/router-options/) that change the behaviour of the `/__health` endpoint:

{{< schema version="v2.8" data="router.json" filter="disable_health,health_path,logger_skip_paths" >}}

### Example: Disable the health endpoint
```json
{
  "version": 3,
  "extra_config": {
    "router": {
      "@comment": "The health endpoint is no longer available",
      "disable_health": true
    }
  }
}
```
### Example: Rename the health endpoint
```json
{
  "version": 3,
  "extra_config": {
    "router": {
      "@comment": "The health endpoint is now under /health instead of /__health",
      "health_path": "/health"
    }
  }
}
```
### Example: Disable the access log of /__health
```json
{
  "version": 3,
  "extra_config": {
    "router": {
      "@comment": "The health endpoint checks do not show in the logs",
      "logger_skip_paths": [
        "/__health"
      ]
    }
  }
}
```

## Custom response formats of the health endpoint
If you'd like to have a health endpoint with a custom response, the simplest solution is to use [stub data](/docs/v2.8/endpoints/static-proxy/) to alter the existing response.

{{< note title="Avoid adding dependencies in your health check" >}}
When setting custom health checks, try not to use external backends connected to databases to determine if KrakenD has to be reloaded or not.
{{< /note >}}

A custom health configuration could look like this:

```json
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
```

In this configuration, KrakenD connects to itself, but instead of returning the content of the internal health endpoint, it sets the `data` defined in the static structure. Notice that the listening `port` in the configuration and the `host` match your deployed KrakenD.

The response content of this custom `/health` endpoint is:

{{< terminal title="k8s check endpoint" >}}
curl http://localhost:8080/health
{"custom": "response","foo": "bar"}
{{< /terminal >}}

The `/__health` endpoint will still be available, but you can rename it, or if your backend does not use it, disable it.