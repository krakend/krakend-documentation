---
lastmod: 2025-01-31
old_version: true
date: 2018-10-29
linktitle: Response caching
title: Caching Strategies
description: Learn how to implement effective caching strategies in KrakenD API Gateway to improve API performance and reduce backend load
weight: 100
menu:
  community_v2.9:
    parent: "060 Request and Response Manipulation"
notoc: false
meta:
  since: v0.4
  source: https://github.com/krakend/krakend-httpcache
  namespace:
  - qos/http-cache
  scope:
  - backend
---
Caching allows you to **store backend responses in memory** to reduce the number of calls a user sends to the origin, **reducing the network traffic and alleviating the pressure on your services**.

KrakenD works similarly to the default rules of a CDN to cache responses, as it adheres mostly to the [RFC-7234](https://datatracker.ietf.org/doc/html/rfc7234) (HTTP/1.1 Caching) in its implementation, and all the internals follow the decisions based on the RFC.

The caching component is a capability of the default KrakenD HTTP client connecting to your upstream services, and can store content in-memory so the next request that is within a valid expiration window can be returned right away without using the network, alleviating pressure and improving times.

KrakenD does not cache the final content delivered to the end-user (the endpoint output), but the response of the backend (close, but not the same). Its approach is to store individual backend responses rather than aggregated content. In an endpoint with aggregation, you will need to add the cache component to all the backends that need caching individually. If you have manipulations of content after retrieving the content, these are not cached either.

{{< note title="Caching increases memory consumption" type="warning">}}
Caching can significantly increase the load and memory consumption as the returned data is saved in memory **until its expiration period**. The size of the cache depends 100% on your backends and configuration. You will need to dimension your instance accordingly, and monitor its consumption!
{{< /note >}}

## What is cached and for how long?
You can only cache your backend's `GET` methods. If you connect to a backend using `POST`, `DELETE`, `PUT`, or any other **unsafe method**, the cache is skipped, and nothing stored. The method that counts for caching or not is the one used to connect to the backend, and not the one used in the endpoint, which might be different. For instance, you could still have an endpoint offering a `POST` method to the end-user, but if the backend uses a `GET` you could cache its response.

When KrakenD receives the response from the backend, checks its headers. **The `Cache-Control` header sets for how long this content is stored** in the cache. See the different headers and values that affect caching below. The cache is stored in a key that contains the final URL sent to the backend plus the combination of any existing `Vary` headers.

When responses are returned to users, the memory is filled according to the header directives. Stale content is replaced by fresh content when needed automatically.

## Cache configuration
To enable the caching of the backend services you only need to add in the `backend` section of your `krakend.json` the following:
```json
{
    "backend": [{
      "url_pattern": "/url-to-cache",
      "host": ["http://host-to-cache.example.com"],
      "extra_config": {
        "qos/http-cache": {}
      }
    }]
}
```

When you don't set any additional parameters, you are creating an individual cache bucket for this backend, with uncapped memory.

A safer option that limits the amount of memory you can set to a backend would be:

```json
{
    "backend": [{
      "url_pattern": "/url-to-cache",
      "host": ["http://host-to-cache.example.com"],
      "extra_config": {
        "qos/http-cache": {
          "@comment": "Allow up to 100 cache entries or ~128MB (bytes not set exactly)",
          "shared": false,
          "max_items": 100,
          "max_size": 128000000
        }
      }
    }]
}
```

The configuration options are as follows:

{{< schema version="v2.9" data="backend_extra_config.json" filter="qos/http-cache" title="Cache options">}}

Notice that `max_size` and `max_items` must coexist. Either you declare none, or you declare both.

## Cache types, cache size and LRU
The caching component allows you to declare how the cache buckets work and their relationship with the hardware. There are two important things to have in mind:

- **Capped or Uncapped memory** decides what you can do with the resources of the host
- **Individual or Shared cache** sets if you want to reuse content in different endpoints

### Capped vs Uncapped memory
**When the memory is capped** (you set `max_items` and `max_size`) you allow every cache bucket to grow up to a defined point. You stablish limits both to the number of entries and the amount of bytes, and enable the LRU algorithm (*Least Recently Used*). The two parameters are required simultaneously (otherwise it falls back to uncapped memory). In this model, when the maximum capacity is exhausted, new cacheable content replaces the old content that has been least recently used (LRU). **This is the safest option**.

**When the memory is uncapped** (you don't set the attributes `max_items` and `max_size`), the content does not have any restriction to store content and can use and exhaust all the memory of the system. This option requires you, the developer, to plan how the cache is going to be used and make numbers. Surpassing the memory limit might set the system unstable or even crash it.

### Individual vs Shared cache
**When the cache is individual** (`shared` property is `false` or missing), every backend definition uses its own cache bucket, and the contents are restricted to the associated backend. It means that other backends or endpoints can't fetch their contents.

**When the cache is shared**, instead of creating the cache bucket isolated in the backend, you use a globally available bucket to all endpoints where other backends that need the same cache key can fetch the contents without needing to ask it again.

This idea is represented in the following example:

![Example of shared and individual cache](/images/documentation/diagrams/cache-options.mmd.svg)

In the diagram above there are four `backend` definitions that could be anywhere across different endpoints. Backends `B` and `C` query their upstream services and store the content in a cache bucket that is constrained to their private access.

On the other hand, backends `A` and `D` use a shared cache bucket, so if one of them saves a result the other can fetch the same cache key without needing to go to the origin.

{{< note title="Shared or individual cache?" type="tip" >}}
Systems with high pressure and concurrency perform better with an individual cache. If the content you are caching is unlikely used often by another endpoint, leave it individual. On the other hand, if several backends are doing the same query repeatedly, you might want to share the cache bucket to improve the performance of your backend systems.
{{< /note >}}




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

## Is my content cached?
KrakenD **does not provide an explicit mechanism for the developer to keep track of hits or misses** or actively manage its entries. For users with advanced needs managing caching who do not want automatic management of entries, we recommend using an advanced caching system such as Varnish Cache. One endpoint might result from several cached components, so the final content might be a sum of caches.

Nevertheless, a simple rule exists to identify whether a response is cached: **look at the response time in the access logs**.

When the content is generated fresh, you'll see in the access log that a response takes a few milliseconds or even seconds, depending on your backend load and performance. But when the content is cached, you'll see times that are at least one order of magnitude smaller (like a few microseconds or even milliseconds). You'll have no doubt when comparing a fresh call to a cached one.

In addition, there are other ways to check if backend responses are served from the cache, like:

- Checking the traces to notice missing backend calls
- Comparing the latency of the requests in your metrics
- Check your backend logs

{{< note title="Configurations not eligible for caching" type="warning" >}}
The components [Client Credentials](/docs/v2.9/authorization/client-credentials/), [Lambda functions](/docs/v2.9/backends/lambda/), [AMQP Consumers or Producers](/docs/v2.9/backends/amqp-consumer/), [Publish/Subscribe](/docs/v2.9/backends/pubsub/), and [HTTP Client plugins](/docs/v2.9/extending/http-client-plugins/) **don't support direct caching** because they use a **custom HTTP client** with extended functionality that does not use the one with caching capabilities. Generally speaking, connections with upstream services that need authentication or custom HTTP clients are not eligible to cache their responses.
{{< /note >}}


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

The [Martian module](/docs/v2.9/backends/martian/) is the component that can transform the headers from the backend as long as you don't use the `no-op` encoding (as it does not allow manipulation). Let's illustrate how you can do this in the following example.

```json
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/v2.9/krakend.json",
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
