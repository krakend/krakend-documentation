---
lastmod: 2023-12-20
date: 2018-10-29
linktitle: Response caching
title: Caching Strategies
description: Learn how to implement effective caching strategies in KrakenD API Gateway to improve API performance and reduce backend load
weight: 100
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
Caching allows you to **store backend responses in memory** to reduce the number of calls a user sends to the origin, reducing the network traffic and alleviating the pressure on your services.

KrakenD's caching approach is to store individual backend responses rather than aggregated content. Although it is a minor implementation detail, it is worth noticing that caching applies to traffic between KrakenD and your microservices, not between the end user and KrakenD.

The caching component adheres to the [RFC-7234](https://datatracker.ietf.org/doc/html/rfc7234) (HTTP/1.1 Caching) in its implementation, and all the internals follow the decisions based on the RFC, requiring you to mark the backends you want to cover by adding the `qos/httpcache` element in the backend section and not much else.

{{< note title="Caching increases memory consumption" type="warning">}}
Caching can significantly increase the load and memory consumption as the returned data is saved in memory **until its expiration period**. The size of the cache depends 100% on your backends. You will need to dimension your instance accordingly, and monitor its consumption!
{{< /note >}}

## What can you cache and for how long?
You can only cache the `GET` method. If you connect to a backend using `POST`, `DELETE`, `PUT`, or any other **unsafe method**, the cache layer is ignored, and nothing is stored. The method that counts is the one used to connect to the backend, and not the one used in the endpoint, which might be different.

The cache key is based on the method, the final URL sent to the backend (the `url_pattern`), plus any additional parameters. The response is stored for the time the `Cache-Control` has defined.

The server sets no limit to the amount of content you will cache, and it is the developer's responsibility to calculate whether the dataset will fit in memory. Memory is filled as cacheable entries are requested. Stale content is replaced by fresh content when needed, but no algorithm allows using a dataset larger than the memory available.

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

There is no additional configuration besides its simple inclusion, although you can pass the `shared` attribute. See an example with a shared cache:

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

The `shared` cache allows different backend definitions with this flag enabled to reuse the store between them when the request is the same. Otherwise, each backend uses a private cache context when the `shared` flag is missing or set to false.

## Cache TTL, size, expiration, purge
When you enable the caching module, your backends control the expiration time of the cache by setting the `Cache-Control` header. If your backends do not set the header or it is set to zero, KrakenD won't store any content in its internal cache.

The response's content size directly impacts KrakenD memory consumption, as KrakenD does not set any hard limit. Therefore, you must be aware of the response sizes, caching times, and the hit rate of the calls.

Finally, KrakenD does not offer any interface to purge the cache. The cache will cleanse itself as defined by the cache-control header, and only a restart of the service would wipe the store entirely. Nevertheless, you can make a few tweaks, as described below.

## Headers affecting the cache
As we said, the only possible methods to cache are `GET` and `HEAD`, but this is only true as long as there isn't a `Range` header in the response (multipart downloads).

The cache takes into account the `Vary` header too. When present, it won't return cached content unless all `Vary` headers match the cached ones.

If a recently generated response is already saved in the cache, it will be retrieved without requiring a connection to the server. However, if the saved response is outdated or stale, any validators included in the new request will be used to give the server an opportunity to respond with a `NotModified` status. If the server provides this status, then the cached response will be returned.

When storing the content in cache, it takes into account the headers `Date`, `Etag` and `Last-Modified`, and KrakenD will send to the `if-none-match` and `if-modified-since` accordingly.

If a response includes both an `Expires` header and a `max-age` directive, the `max-age` directive overrides the `Expires` header, even if the `Expires` header is more restrictive.

The `Cache-Control` header honors the time settings and the properties `no-store`, `only-if-cached` , `no-cache`.

## Custom directives (Stale cache, freshness)
You can allow your API consumers to pass **custom directives (instructions)** in their `Cache-Control` header when they make the request.

KrakenD can honor the directives `max-stale`, `max-age`, `stale-if-error`, and `min-fresh` present in the request when you add the `Cache-Control` in the `input_headers` endpoint configuration:

```json
{
 "endpoint": "/cached-or-not",
 "input_headers": ["Cache-Control"]
}
```

If you don't allow the input headers to pass, KrakenD ignores the consumer's request and returns the cached content according to the backend response.

But if you do, if clients are willing to accept **stale cache** and pass the directive `max-stale` (or similarly `stale-if-error`) in the request, then a response that has exceeded its expiration time by no more than the specified number of seconds in the directive receives a stale cache.

### Overriding the expiration time
As you have seen, the caching module does not accept any parameters to control the cache expiration because it relies on the input headers it finds when the response returns. But as KrakenD can transform data in many ways, you can modify the `Cache-Control` header right before the cache module picks it.

The [Martian module](/docs/backends/martian/) is the component that can transform the headers from the backend as long as you don't use the `no-op` encoding (as it does not allow manipulation). Let's illustrate how you can do this in the following example.

```json
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/v2.6/krakend.json",
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
