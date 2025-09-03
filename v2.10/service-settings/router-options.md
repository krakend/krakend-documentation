---
lastmod: 2023-11-02
old_version: true
date: 2021-10-29
linktitle: Router options
title: Router Options
description: Explore the router options available in KrakenD API Gateway to customize the routing behavior and optimize API request handling
weight: 120
notoc: false
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
  community_v2.10:
    parent: "040 Routing and Forwarding"
---

The **optional router configuration** allows you to set global flags that change how KrakenD processes the requests globally at the router layer.

Generally speaking **you don't need this**. But there is an exception in every case, and you might need to change some values.

## Configuration for the router

The `router` controls the behavior of KrakenD toward users. Its settings affect all activity in the gateway. For instance, you can **obfuscate the X-KrakenD-version header**, set a **custom body for 404 errors**, **remove the requests from the logs**, or define how to **calculate client IPs** to name a few examples.

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

{{< schema version="v2.10" data="router.json" >}}


{{< note title="Caution with `disable_redirect_fixed_path`" type="error" >}}
This flag can lead to the malfunctioning of your router. If your API configuration has paths that could collide, leave its value with the **safe choice** `disable_redirect_fixed_path=true` to avoid possible panics.
{{< /note >}}

## Return the real client IP
The flags `forwarded_by_client_ip`, `remote_ip_headers`, and `trusted_proxies` determine together how you get the client IP address.

{{< note title="Dependant flags" type="info" >}}
When your setup requires any attributes `forwarded_by_client_ip` or `trusted_proxies`, you **must declare both**. The attribute `remote_ip_headers` is only necessary when you want to overwrite the default headers.
{{< /note >}}

### How the IP is retrieved internally
The flow the gateway follows to extract the client IP is as follows:

- The gateway fetches the IP from the connecting remote address
- Then it checks the IP from the headers listed under `remote_ip_headers`. When this list is not in the configuration, it looks in  `X-Forwarded-For` and `X-Real-IP` (default lookup headers).
- Unless you have a single KrakenD exposed to the internet or working locally, the client request will travel through different relays. These relays generally modify the header to append where they received the request from (whether it is the originating client, a load balancer, another proxy, etc.). So, you usually have a comma-separated list of IPs in the header containing the IP as they travel from one relay to the other. For instance, if there is just one hop before KrakenD, the header the gateway sees could look like `X-Forwarded-For: 1.2.3.4,172.20.0.1` where `1.2.3.4` is the real IP of the user, and `172.20.0.1` the last relay seen.
- The list of `trusted_proxies` sets which IPs are part of these network hops, and **the last IP before a known trusted proxy** is the actual IP.
- If checking the trusted proxies does not work, it will return the remote address in the first step.

Here's an example of behavior. Suppose the gateway receives a header `X-Forwarded-For: A,B,C,D` (IPs are expressed as letters for simplification). If your `trusted_proxies` configuration contains ranges for `C` and `D`, then the returned IP is `B`, as `A` could have been spoofed by the client.

**The real IP is stored in the `X-Forwarded-For` header that KrakenD uses.**

The following example shows a configuration that takes the user IP from an `X-Forwarded-For` header only, and the network origin has relays in the range `172.16.0.1/12` (IPv4 Private Address Space). The endpoint `/ip` returns the IP received. You can test this locally:

```json
{
  "$schema": "https://www.krakend.io/schema/v2.10/krakend.json",
  "version": 3,
  "echo_endpoint": true,
  "extra_config": {
      "router":{
          "forwarded_by_client_ip": true,
          "remote_ip_headers": [
            "X-Forwarded-For"
          ],
          "trusted_proxies": [
            "172.16.0.1/12"
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

## Hide the version in the `X-KrakenD-version` header
You can remove the KrakenD version your installation uses by adding the `hide_version_header` flag as follows:

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

## Custom JSON body for 404 and 405 errors
You can also define generic responses for 404 and 405 errors by defining the response object you will return to clients using the `error_body` property. Here is an example:

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

## Returning the gateway error message
The secure choice of KrakenD is that all errors generated at the gateway **are not returned to the client in the body**. By setting `return_error_msg` (*boolean*) to `true`, when there is an error in the gateway (such as a timeout, a non-200 status code, etc.), it returns the client the reason for the failure. The error is **written in the body as is**.

{{< note title="Gateway errors != backend errors" >}}
This option will return the gateway interpretation of the error (such as there was an *invalid status code*). Nevertheless, when this option is mixed with a backend flag [`return_error_code`](/docs/v2.10/backends/detailed-errors/#showing-backend-errors) set to `true`, the backend error is returned, but its encoding is lost.

Consider using [`return_error_details`](/docs/v2.10/backends/detailed-errors/#return-backend-errors-in-a-new-key) to create a key with the error and support aggregation, or just use `no-op` encoding if you need errors as returned by the backend.
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

## Remove requests from logs
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
