---
lastmod: 2021-05-02
date: 2018-10-29
linktitle: Caching responses
title: Caching backend responses
weight: 50
menu:
  community_current:
    parent: "050 Backends Configuration"
notoc: true
meta:
  since: 0.4
  source: https://github.com/devopsfaith/krakend-httpcache
  namespace:
  - qos/http-cache
  scope:
  - backend
---

Sometimes you might want to reuse a previous response of a backend instead of asking for the same information over the network again. In this cases, it is possible to enable **in-memory** caching for the desired backend responses.

This caching technique applies to traffic between KrakenD and your microservices endpoints only and is not a caching system for the end-user endpoints. To enable the cache, you only need to add in the configuration file the `qos/httpcache` middleware.

When enabled all connections for the configured backend are cached in-memory. The cache content is based on the response for the final URL sent to the backend (the `url_pattern` plus any additional parameters). The response is stored for the time the `Cache-Control` has defined and there is no way to purge it externally.

{{< note title="Performance notice!" type="error">}}
This option can increase the load and memory consumption heavily as KrakenD needs to keep in memory all the returned data during the expiration period. Only the backend has control over this. Use it wisely and monitor its consumption!
{{< /note >}}

If you enable this module, you are required to be very aware of the response sizes, caching times and the hit-rate of the calls.

Enable the caching of the backend services in the `backend` section of your `krakend.json` with the middleware:
{{< highlight json >}}
{
  "extra_config": {
    "qos/http-cache": {}
  }
}
{{< /highlight >}}


The middleware **does not require additional configuration** other than its simple inclusion, although the `shared` attribute can be passed.

See an example (with shared cache):

With shared cache:
{{< highlight json >}}
{
    "endpoint": "/cached",
    "backend": [{
      "url_pattern": "/",
      "host": ["http://my-service.tld"],
      "extra_config": {
        "qos/http-cache": {
            "shared": true
        }
      }
    }]
}
{{< /highlight >}}

The `shared` cache makes that different backend definitions with this flag enabled can reuse the cache. When the `shared` flag is missing or set to false, the backend uses its own cache private context.
