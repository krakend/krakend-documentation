---
lastmod: 2023-06-01
date: 2018-10-29
linktitle: Caching responses
title: Caching Strategies in KrakenD API Gateway
description: Learn how to implement effective caching strategies in KrakenD API Gateway to improve API performance and reduce backend load
weight: 50
menu:
  community_current:
    parent: "060 Request and Response Manipulation"
notoc: true
meta:
  since: 0.4
  source: https://github.com/krakend/krakend-httpcache
  namespace:
  - qos/http-cache
  scope:
  - backend
---
Caching allows you to **store backend responses in memory** to reduce the number of calls a user sends to the origin, reducing the network traffic and alleviating your services' pressure.

KrakenD's caching approach is to store individual backend responses rather than aggregated content. Although it is a minor implementation detail, it is worth noticing that caching applies to traffic between KrakenD and your microservices, not between end-user and KrakenD.

The caching component is practically a flag, requiring you to mark the backends you want to cover by adding the `qos/httpcache` element in the backend section.

When enabled, all responses from safe methods (`GET`, or `HEAD`) are cached in-memory. The cache key is based on the method, the final URL sent to the backend (the `url_pattern`) plus any additional parameters. The response is stored for the time the `Cache-Control` has defined.

If you connect to a backend using `POST`, `DELETE`, `PUT`, or any other **unsafe method**, the cache layer is ignored and nothing is stored.

When there is a `Range` header in the response, the caching layer is ignored too.

{{< note title="Caching increases memory consumption" type="warning">}}
Caching can significantly increase the load and memory consumption as the returned data is saved in memory **until its expiration period**. The size of the cache depends 100% on your backends. You will need to dimension your instance accordingly, and monitor its consumption!
{{< /note >}}

## Cache configuration
Enable the caching of the backend services in the `backend` section of your `krakend.json` with the middleware:
{{< highlight json "hl_lines=6">}}
{
    "backend": [{
      "url_pattern": "/url-to-cache",
      "host": ["http://host-to-cache"],
      "extra_config": {
        "qos/http-cache": {}
      }
    }]
}
{{< /highlight >}}

There is no additional configuration besides its simple inclusion, although you can pass the `shared` attribute. See an example with shared cache:

```json
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
```

The `shared` cache makes that different backend definitions with this flag enabled can reuse the cache between them when the request is the same. Otherwise, each backend uses a private cache context when the `shared` flag is missing or set to false.

## Cache TTL, size, expiration, purge
When you enable the caching module, your backends control the expiration time of the cache by setting the `Cache-Control` header. If your backends do not set the header or it is set to zero, KrakenD won't store any content in its internal cache.

The response's content size directly impacts KrakenD memory consumption, as KrakenD does not set any hard limit. Therefore, you must be aware of the response sizes, caching times, and the hit rate of the calls.

Finally, KrakenD does not offer any interface to purge the cache. The cache will cleanse itself as defined by the cache-control header, and only a restart of the service would wipe the store entirely. Nevertheless, you can make a few tweaks, as described below.

### Overriding the expiration time
As you have seen, the caching module does not accept any parameters to control the cache expiration because it relies on the input headers it finds when the response returns. But as KrakenD can transform data in many ways, you can modify the `Cache-Control` header right before the cache module picks it.

The [Martian module](/docs/backends/martian/) is the component that can transform the headers from the backend as long as you don't use the `no-op` encoding (as it does not allow manipulation). Let's illustrate how you can do this in the following example.

```json
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/v2.5/krakend.json",
    "endpoints": [
        {
            "endpoint": "/cached",
            "backend": [
                {
                    "host": ["http://worldtimeapi.org/"],
                    "url_pattern": "/api/timezone/Europe/Madrid",
                    "extra_config": {
                        "qos/http-cache": {
                            "@comment": "This API returns a cache-control: max-age=0 so KrakenD won't cache this unless changed"
                        },
                        "modifier/martian": {
                            "header.Modifier": {
                                "scope": ["response"],
                                "name": "Cache-Control",
                                "value": "max-age=60, public",
                                "@comment": "We will change the max-age policy before KrakenD checks the content for caching. Now content is cached 60 seconds."
                              }
                        }
                    }
                }
            ]
        }
    ]
}
```

Notice in the configuration above how KrakenD consumed a backend without an actionable cache-control header, but it set a cache of one minute anyway. This technique might be handy when you have little control of the cache headers in the response.
