---
lastmod: 2023-01-12
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
  community_v2.1:
    parent: "030 Service Settings"
---

The **optional router configuration** allows you to set global flags that change how KrakenD processes the requests globally at the router layer.

Generally speaking **you don't need this**. But in every case, there is an exception, and you might need to change some values.

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

{{< schema version="v2.1" data="router.json" >}}


{{< note title="Caution with `disable_redirect_fixed_path`" type="error" >}}
This flag can lead to the malfunctioning of your router. If your API configuration has paths that could collide, leave its value with the **safe choice** `disable_redirect_fixed_path=true` to avoid possible panics.
{{< /note >}}

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
When the flag is set to `true`, the banner header will show an `undefined` version. To remove the header entirely, you must remove it in the CDN or layer in front of KrakenD.

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
This option does not relate to the body of your backend errors. If you are looking for this option, see [return detailed errors](/docs/v2.1/backends/detailed-errors/#showing-backend-errors) instead.
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

## Example: Obtaining the real IP
The flags - `forwarded_by_client_ip`, `remote_ip_headers`, and `trusted_proxies` determine how to get the client IP address (read its documentation above)

The following example shows a configuration that takes the user IP from the `X-Custom-Ip` header only when the network origin is `172.20.0.1/16`:

```json
{
  "version": 3,
  "extra_config": {
      "router":{
          "forwarded_by_client_ip": true,
          "remote_ip_headers": [
            "X-Custom-Ip"
          ],
          "trusted_proxies": [
            "172.20.0.1/16"
          ]
      }
}
```

## Example: Remove requests from logs
There are two options to remove content from logs, the `logger_skip_paths` (list of paths you don't want to see in the logs), and `disable_access_log`, which stops registering access requests.

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