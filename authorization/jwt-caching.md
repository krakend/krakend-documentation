---
lastmod: 2023-01-17
date: 2018-11-03
linktitle: JWK key caching
title: Shared JWK caching
description: A good JWK URL caching strategy reduces the impact KrakenD does to your identity provider while you guarantee
weight: 21
menu:
  community_current:
    parent: "060 Authentication & Authorization"
meta:
  since: 2.3
  source: https://github.com/krakendio/krakend-jose
  namespace:
  - auth/jwk-client
  scope:
  - service
  log_prefix:
  - "[SERVICE: JWK Client]"
---
The [JWT validation](/docs/authorization/jwt-validation/) and [JWT signing](/docs/authorization/jwt-signing/) components do not apply any type of cache by default.

Validating tokens in a high-throughput scenario can be a consuming operation. In order to refrain the gateway from downloading the signing keys on each request you can enable caching. It's usually a bad idea to not cache the content of the JWK URL as your identity provider would receive a huge amount of traffic.

There are **two levels** of cache you can apply to the signing keys, designed to support the most extreme conditions:

- Per-endpoint cache
- Client cache (shared by all endpoints)

The `jwk_url` is a location that provides the key definition that allows KrakenD to determine if a token is legitimate or not. Unless the content of this URL changes (key rotation), there is no need to continously retrieve it.

## What happens when you don't use caching
If for instance you have 1000 endpoints in your configuration, when KrakenD starts, your identity server will receive an initial blast of 1000 endpoints requesting its JWK URL for the first time. In addition, each request to an endpoint, will generate another hit to your identity server.

An identity server is not designed to support a high pressure like an API gateway can, and you should therefore, limit the interaction between the gateway and the identity provider by implementing the one or two possible caching layers.

## Per-endpoint JWK cache
Whenever an endpoint requires JWT validation, you can (and you should) enable its `cache` and `cache_duration` properties. Otherwise, **every request to the endpoint creates a request to the identity server**.

You can configure per-endpoint cache using the following options in the `auth/validator`:


{{< schema data="auth/validator.json" filter="cache,cache_duration" >}}

## Per-service JWK cache
If you have a lot of endpoints, each of them will need to retrieve the JWK URL to cache it. You can add a per-service JWK cache by adding cache to the connecting client. This option goes at the service level.

Combining the two you would have a configuration like this:

```json
{
    "version": 3,
    "extra_config": {
        "auth/jwk-client": {
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

Each endpoint will cache the URL for 1 hour, but when requesting upon expiration, the cache client will make sure to retrieve from the original source maximum every 15 minutes.