---
lastmod: 2025-09-03
date: 2025-09-03
linktitle: HTTP Per-backend Client settings
title: "HTTP Per-backend Client settings"
description: "The HTTP client options allow you to configure the behavior to connect to your backends using HTTP"
weight: 145
notoc: true
menu:
  community_current:
    parent: "040 Routing and Forwarding"
meta:
  since: 2.11
  namespace:
  - backend/http/client
  scope:
  - backend
  log_prefix:
  - "[BACKEND: /foo][backend/http/client]"
---
The HTTP client namespace allows you to set the behavior of the HTTP connections between KrakenD and your backend service.

### Send the payload on 307 and 308 redirects
KrakenD does not duplicate the body of the request when following a redirection because automatically doing it would affect the performance of all requests. In the unusual cases where your backend responds with a `307 Temporary Redirect` or a `308 Permanent Redirect`, enable the following flag to resend the original payload to the final redirected service:

{{< schema data="backend/http_client.json" filter="enable_redirect_post" >}}

Here is a configuration example:

```json
{
  "version": 3,
  "$schema": "https://www.krakend.io/schema/v{{< product minor_version >}}/krakend.json",
  "endpoints": [
    {
      "endpoint": "/foo",
      "backend": [
        {
          "host": ["https://api"],
          "url_pattern": "/url-that-will-redirect-with-307",
          "extra_config": {
            "backend/http/client": {
              "enable_redirect_post": true
            }
          }
        }
      ]
    }
  ]
}
```

## Avoid HTTP redirection and other options
To prevent KrakenD from following redirects, use specific TLS options, and utilize intermediate proxies and other options, refer to the [HTTP Client options in the Enterprise Edition](/docs/enterprise/backends/http-client/).