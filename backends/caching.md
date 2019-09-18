---
lastmod: 2018-10-29
date: 2018-10-29
linktitle: Caching responses
title: Caching backend responses
weight: 50
source: https://github.com/devopsfaith/krakend-httpcache
menu:
  documentation:
    parent: backends
---

Sometimes you might want to reuse a previous response of a backend instead of asking for the same information over the network again. In this cases, it is possible to enable **in-memory** caching for the desired backend responses.

This caching technique applies to traffic between KrakenD and your microservices endpoints only and is not a caching system for the end-user endpoints. To enable the cache, you only need to add in the configuration file the `httpcache` middleware.

When enabled all connections for the configured backend are cached in-memory for the amount of time retrieved from the `Cache-Control` header of the response.

{{% note title="Performance notice!" %}}
This option can increase the load and memory consumption heavily as KrakenD needs to keep in memory all the returned data during the expiration period. Use wisely!
{{% /note %}}

If you enable this module, you are required to be very aware of the response sizes, caching times and the hit-rate of the calls.

Enable the caching of the backend services in the `backend` section of your `krakend.json` with the middleware:

    "github.com/devopsfaith/krakend-httpcache": {}

The middleware does not require additional configuration other than the simple inclusion.

See an example:

    ...
    "backend": [
    {
      "url_pattern": "/",
      "host": ["http://my-service.tld"],
      "extra_config": {
        "github.com/devopsfaith/krakend-httpcache": {}
      }
    }
    ]
...
