---
lastmod: 2023-03-17
date: 2023-03-17
linktitle: JWK key caching
title: Shared JWK Caching for Authorization
description: Implement shared JWK caching for efficient authorization in KrakenD. Learn how to cache and manage JSON Web Key (JWK) sets to optimize authorization's performance.
weight: 21
menu:
  community_current:
    parent: "080 Authentication & Authorization"
meta:
  since: 2.3
  source: https://github.com/krakend/krakend-jose
  namespace:
  - auth/validator
  scope:
  - service
  log_prefix:
  - "[SERVICE: JOSE]"
---
The [JWT validation](/docs/authorization/jwt-validation/) and [JWT signing](/docs/authorization/jwt-signing/) components do not apply cache by default.

Validating tokens in a high-throughput scenario can be a consuming operation. To refrain the gateway from downloading on each request the signing keys, you can enable caching. It's usually a bad idea to not cache the content of the JWK URL as your identity provider would receive a huge amount of traffic.

{{< note title="Caching does not apply to `jwk_local_path`" type="info" >}}
When instead of using `jwk_url` you have the keys on disk and you use `jwk_local_path`, then you don't need to set any cache at all.
{{< /note >}}


## What happens when you don't use caching
If, for instance, you have 1000 endpoints in your configuration, when KrakenD starts, your identity server(s) will receive an initial blast of 1000 connections requesting their corresponding JWK URL for the first time. In addition, each request to a secure endpoint will generate another hit to your identity server.

An identity server has another function and is not designed to support high pressure like an API gateway. Plus, as you can see, there is no point in stressing it to retrieve the same content repeatedly. Therefore, you should limit the interaction between the gateway and the identity provider by implementing one or two possible caching layers.

## Enabling JWK URL caching
There are **two levels** of cache you can apply designed to support together the most extreme conditions:

- Per-endpoint cache (`cache` on the `auth/validator` in the endpoint)
- Shared JWK cache between all endpoints ( `shared_cache_duration` on the `auth/validator` at the service level)

We encourage you to configure at least a per-endpoint caching, but adding a client cache will offer you even more control over the traffic you send to the identity provider(s).

The caching works for the `jwk_url`, an HTTP-based location that contains the signing keys and allows KrakenD to determine whether a token is legitimate. Therefore, when the content of this URL changes (you are doing **key rotation**), the gateway needs to use a TTL that is compatible.

### Per-endpoint JWK cache
Whenever an endpoint requires JWT validation, you should always enable its `cache` and `cache_duration` properties. Otherwise, **every request to the endpoint creates a request to the identity server**.

You can configure per-endpoint cache using the following options in the `auth/validator`:

{{< schema data="auth/validator.json" filter="cache,cache_duration" >}}

If you want that several endpoints consuming the same `jwk_url` share a unique cache entry, then you must enable a per-service cache as explained below.

### Shared JWK cache
If you have a lot of endpoints, each of them will need to retrieve the JWK URL to cache it. You can add a shared JWK cache where one endpoint call will enable the cache for another one using the same origin URL. This option is not configured in the endpoint, but at the service level.

The shared cache requires the following configuration, plus **having `cache` in all desired endpoints**:

{{< schema data="auth/jose.json" >}}

{{< note title="The `cache` flag is still required" type="info" >}}
**Important**: Setting the flag alone does not work unless you add at least one endpoint with the `cache` flag set to `true`.
{{< /note >}}

Example:

```json
{
    "version": 3,
    "extra_config": {
        "auth/validator": {
            "@comment": "Enable a JWK shared cache amongst all endpoints of 15 minutes",
            "shared_cache_duration": 900
        }
    },
    "endpoints": [
    {
      "endpoint": "/protected-1",
      "extra_config": {
        "auth/validator": {
            "cache": true,
            "cache_duration": 3600,
            "jwk_url": "https://your-id-provider",
            "@comment": "Rest of the validator options omitted for simplicity"
        }
      }
    }
    ]
}
```


### Cache recommendations
**Combining the two levels of cache is usually the ideal scenario**. On one side, each endpoint has a scoped local cache entry and is the perfect strategy to handle contention. On the other side, adding the JWK client cache makes that when each endpoint cache expires, can stll rely on a more global level of cache which allows you to control the exact time between requests to your identity server.

You might thing that one global cache level would be enough, but having this granularity makes the system way more efficient, performant, and error-free.

In all, you would have a configuration like this:

```json
{
    "version": 3,
    "extra_config": {
        "auth/validator": {
            "@comment": "Enable a JWK shared cache amongst all endpoints of 15 minutes",
            "shared_cache_duration": 900
        }
    },
    "endpoints": [
    {
      "endpoint": "/protected-1",
      "extra_config": {
        "auth/validator": {
            "cache": true,
            "cache_duration": 3600,
            "jwk_url": "https://your-id-provider",
            "@comment": "Rest of the validator options omitted for simplicity"
        }
      }
    },
    {
      "endpoint": "/protected-2",
      "extra_config": {
        "auth/validator": {
            "cache": true,
            "cache_duration": 3600,
            "jwk_url": "https://your-id-provider",
            "@comment": "Rest of the validator options omitted for simplicity"
        }
      }
    }
    ]
}
```
In the example above, each endpoint will cache the JWK URL for 1 hour, but when requesting again upon expiration, the cache client will return its cached content at a pace of 1 request every 15 minutes (900 seconds).