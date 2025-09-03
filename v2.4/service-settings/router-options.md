---
lastmod: 2023-09-25
old_version: true
date: 2021-10-29
linktitle: Router options
title: Customizing router behavior
weight: 10
notoc: true
meta:
  since: v2.0
  source: https://github.com/krakend/krakend-cors
  namespace:
  - router
  scope:
  - service
  log_prefix:
  - "[SERVICE: Gin]"

menu:
  community_v2.4:
    parent: "030 Service Settings"
---

The **optional router configuration** allows you to set global flags that change how KrakenD processes the requests globally at the router layer.

Generally speaking **you don't need this**. But there is an exception in every case, and you might need to change some values.

## Configuration for the router

The `router` controls the behavior of KrakenD toward users. Its settings affect all activity in the gateway. For instance, you can **obfuscate the X-KrakenD-version header**, set a **custom body for 404 errors**, or **remove the requests from the logs**, to name a few examples.

To change the router behavior, you must add the namespace `router` inside the `extra_config` at the root of the configuration file. For instance:

```json
{
  "version": 3,
  "extra_config": {
    "router": {
       "hide_version_header": true
    }
  }
}
```
All the options you can set under `router` are:

{{< schema version="v2.4" data="router.json" >}}


{{< note title="Caution with `disable_redirect_fixed_path`" type="error" >}}
This flag can lead to the malfunctioning of your router. If your API configuration has paths that could collide, leave its value with the **safe choice** `disable_redirect_fixed_path=true` to avoid possible panics.
{{< /note >}}

## Return the real client IP
The flags - `forwarded_by_client_ip`, `remote_ip_headers`, and `trusted_proxies` determine how you get the client IP address.

The flow the gateway follows to extract the client IP is as follows:

- The gateway fetches the IP from the connecting remote address
- Then it checks the IP from the headers listed under `remote_ip_headers`, or from `X-Forwarded-For` or `X-Real-IP` when there isn't such a list.
- When the request travels through different relays (unless you have a single KrakenD exposed to the internet is usually the case), these relays generally modify the header to append where they received the request from (whether it is the originating client, a load balancer, another proxy, etc.). So, you usually have a comma-separated list of IPs in the header containing the IP as they travel from one relay to the other. For instance, if there is just one hop before KrakenD, the header the gateway sees could look like `X-Forwarded-For: 1.2.3.4,172.20.0.1` where `1.2.3.4` is the real IP of the user, and `172.20.0.1` the last relay seen.
- When an IP travels through the relays, the list of `trusted_proxies` sets which IPs are part of these network hops, and **the last IP before a known trusted proxy** is the actual IP.
- If checking the trusted proxies does not work, it will return the remote address in the first step.

Here's an example of behavior. Suppose the gateway receives a header `X-Forwarded-For: A,B,C,D` (IPs are expressed as letters for simplification). If your `trusted_proxies` configuration contains ranges for `C` and `D`, then the returned IP is `B`, as `A` could have been spoofed by the client.

**The real IP is stored in the `X-Forwarded-For` header that KrakenD uses.**

The following example shows a configuration that takes the user IP from a `X-Forwarded-For` header only, and the network origin has relays in the range `172.20.0.1/16`. The endpoint `ip` returns the IP received, you can test this locally:

```json
{
  "$schema": "https://www.krakend.io/schema/v2.3/krakend.json",
  "version": 3,
  "echo_endpoint": true,
  "extra_config": {
      "router":{
          "forwarded_by_client_ip": true,
          "remote_ip_headers": [
            "X-Forwarded-For"
          ],
          "trusted_proxies": [
            "172.20.0.1/16"
          ]
      }
  },
  "endpoints": [
    {
      "endpoint": "/ip",
      "backend": [
        {
          "host": ["http://localhost:8080"],
          "url_pattern": "/__echo",
          "allow": ["req_headers.X-Forwarded-For"]
        }
      ]
    }
  ]
}
```

## Example: Hide the version in the `X-KrakenD-version` header
```json
{
  "version": 3,
  "extra_config": {
    "router": {
       "hide_version_header": true
    }
  }
}
```
The banner header will show an `undefined` version when the flag is set to `true`. To remove the header entirely, you must remove it in the CDN or layer in front of KrakenD.

## Example: Custom JSON body for 404 and 405 errors
```json
{
  "version": 3,
  "extra_config": {
    "router": {
      "error_body": {
        "404": {
          "msg": "Unknown endpoint",
          "status": 404
        },
        "405": {
          "oh-my-god": "What on earth are you requesting?"
        }
      }
    }
  }
}
```

## Example: Returning the gateway error message
The secure choice of KrakenD is that all errors generated at the gateway **are not returned to the client in the body**. By setting `return_error_msg` (*boolean*) to `true`, when there is an error in the gateway (such as a timeout, a non-200 status code, etc.), it returns the client the reason for the failure. The error is **written in the body as is**.

{{< note title="Gateway errors != backend errors" >}}
This option does not relate to the body of your backend errors. If you are looking for this option, see [return detailed errors](/docs/v2.4/backends/detailed-errors/#showing-backend-errors) instead.
{{< /note >}}

```json
{
  "version": 3,
  "extra_config": {
      "router":{
          "return_error_msg":true
      }
}
```

## Example: Remove requests from logs
There are two options to remove content from logs: the `logger_skip_paths` (list of paths you don't want to see in the logs) and `disable_access_log`, which stops registering access requests.

```json
{
  "version": 3,
  "extra_config": {
      "router":{
          "logger_skip_paths":[
            "/__health"
          ],
          "disable_access_log": true
      }
}
```