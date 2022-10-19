---
lastmod: 2022-01-19
date: 2021-10-29
linktitle: Router options
title: Customizing router behavior
weight: 10
notoc: true
meta:
  since: 2.0
  source: https://github.com/krakendio/krakend-cors
  namespace:
  - router
  scope:
  - service
  log_prefix:
  - "[SERVICE: Gin]"

menu:
  community_current:
    parent: "030 Service Settings"
---

The **optional router configuration** allows you to set global flags that change the way KrakenD processes the requests at the router layer.

Generally speaking **you don't need this**. But in every case there is an exception and you might eventually need to change some value.

## Configuration for the router

To change the router behavior, add the namespace `router` under the global `extra_config`, and set one or more flags as depicted below:

{{< schema data="router.json" >}}


{{< note title="Caution with `disable_redirect_fixed_path`" type="error" >}}
This flag can lead to malfunctioning of your router. If your API configuration has paths that could collide leave its value with the **safe choice** `disable_redirect_fixed_path=true` to avoid possible panics.
{{< /note >}}


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
The secure choice of KrakenD is that all errors generated at the gateway **are not returned to the client in the body**. By settting `return_error_msg` (*boolean*) to `true`, when there is an error in the gateway (such as a timeout, a non-200 status code, etc.) it returns to the client the reason for the failure. The error is **written in the body as is**.

{{< note title="Gateway errors != backend errors" >}}
This option does not relates to the body of your backend errors. If you are looking for this option see [return detailed errors](/docs/backends/detailed-errors/#showing-backend-errors) instead.
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
There are two options to remove content from logs, the `logger_skip_paths` (list of paths you don't want to see in the logs), and `disable_access_log` which stops registering access requests.

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