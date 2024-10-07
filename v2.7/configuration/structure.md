---
lastmod: 2023-01-31
old_version: true
date: 2018-09-21
linktitle: The configuration file
title: Configuration Structure
description: Learn about the configuration structure and organization in KrakenD API Gateway to effectively define the behavior of your API gateway
menu:
  community_v2.7:
    parent: "010 Configuration files"
weight: 10
---
All KrakenD behavior depends on its configuration file(s). You'll find it referenced through all this documentation and for simplicity as the `krakend.json`, although the configuration [supports formats other than JSON](/docs/v2.7/configuration/supported-formats/) and it can be described [using multiple files and templates](/docs/v2.7/configuration/flexible-config/). Being familiar with its structure it's essential.

The correctness of a configuration file is determined by the [check](/docs/v2.7/configuration/check/) and [audit](/docs/v2.7/configuration/audit/) commands using different perspectives.

## Configuration file structure
There are a large number of options you can put in this file. Let's focus now only on the main structure:
```json
{
    "$schema": "https://www.krakend.io/schema/v2.7/krakend.json",
    "version": 3,
    "endpoints": [],
    "extra_config": {}
}
```


- `$schema`: *Optional*. When added, enables [IDE integration](/docs/v2.7/developer/ide-integration/) with autocompletion and documentation. Defines the JSON schema to validate your configuration. Is used by ` krakend check --lint`.
- `version` (*mandatory*): The version of the configuration file format (not the version of KrakenD).
  - Format version `3`: **Current** (since `v2.0`)
  - Format version `2`: Deprecated in 2022, for versions between `v0.4` and `v1.4.1`
  - Format version `1`: Deprecated in 2016, for versions `v0.3.9` and older.
- `endpoints[]`: An array of [endpoint objects](/docs/v2.7/endpoints/) offered by the gateway and all the associated backends and configurations. This is your API definition.
- `extra_config{}`: Service components' configuration. Whatever is not a core functionality of the [Lura Project](https://luraproject.org) is declared in a unique **namespace** (a key) in the configuration, so that you can configure multiple elements without collisions.

### The `endpoints` structure
Inside the `endpoints`, you declare an array with [endpoint objects](/docs/v2.7/endpoints/). Every object has an `endpoint` (the URL) the gateway offers to users. For each endpoint, you need to declare at least one `backend` (the data origin).

It looks like this:

```json
{
    "endpoints": [
        {
          "endpoint": "/v1/foo-bar",
          "backend": [
            {
              "url_pattern": "/foo",
              "host": [
                "https://my.foo-api.com"
              ]
            },
            {
              "url_pattern": "/bar",
              "host": [
                "https://my.bar-api.com"
              ]
            }
          ]
        }
      ]
}
```

The configuration above declares an endpoint `/v1/foo-bar`, which returns the result of merging the responses from two backends `https://my.foo-api.com/foo` and `https://my.bar-api.com/bar`. These two calls execute in parallel.

That's the basic structure of endpoints; for more information see [how to create endpoints](/docs/v2.7/endpoints/).

### The `extra_config` structure
KrakenD is very modular and comes bundled with many components that extend the core functionality of the [Lura Project](https://luraproject.org). The `extra_config` stores each component configuration that is not handled by Lura itself.

Components declare in their source code a **unique namespace**. KrakenD registers the component during the startup, and it passes to the component the configuration found under a key matching the **namespace** inside the `extra_config` object.


```json
    {
        "extra_config": {
          "component-1-namespace": {
            "some": "config"
          },
          "component-2-namespace": {
          }
        }
    }
```

For instance, the [extended logging component](/docs/v2.7/logging/) uses the **namespace** `telemetry/logging`:

```json
{
    "version": 3,
    "extra_config": {
        "telemetry/logging": {
          "level": "WARNING",
          "prefix": "[KRAKEND]",
          "stdout": true
        }
    }
}
```

### Placements for the `extra_config`
The `extra_config` can appear in the root of the file and on other placements (or levels) as well. It depends entirely on **the scope** of every component and the nature of its functionality.

An `extra_config` in the **root level** usually sets functionalities with a **service scope**: these influence the gateway globally and on every request (e.g., metrics). On the other hand, `extra_config` placed more profound in the configuration affects a tinier scope. An example could be a configuration that is loaded when a certain endpoint is called.

All components will seek the `extra_config` in its defined scope. The possible placements of the `extra_config` are:

- `service` (root level)
- `endpoint`
- `backend`

For instance, you might want to set a [rate limit](/docs/v2.7/throttling/) between a user and a `/my-rate-limited` endpoint in KrakenD. And for that, you would place the `extra_config` inside that `endpoint` scope. Or you might want to limit the connections between a KrakenD endpoint against your services; then you would place the `extra_config` in the `backend` scope.

**You don't have to guess where to put the `extra_config`**. Each component has in the documentation what is the scope(s) is built for.

### Example
The following code is an example defining two simultaneous [rate limiting strategies](/docs/v2.7/throttling/): A limit of 5000 reqs/second for a specific endpoint, but yet, one of its backends accepts a maximum of 100 reqs/s. When the backend limit is reached, the user will have partial responses, and when both are surpassed the user won't have data from any of the backends.

Notice how `extra_config` is present in the endpoints and backend scopes.

```json
{
  "version": 3,
  "endpoints": [
    {
      "endpoint": "/limited-to-5000-per-second",
      "extra_config": {
        "qos/ratelimit/router": {
          "max_rate": 5000
        }
      },
      "backend": [
        {
          "host": [
            "http://slow.backend.com/"
          ],
          "url_pattern": "/slow/endpoint",
          "extra_config": {
            "qos/ratelimit/proxy": {
              "max_rate": 100,
              "capacity": 1
            }
          }
        },
        {
          "host": [
            "http://fast.backend.com/"
          ],
          "url_pattern": "/fast/endpoint"
        }
      ]
    }
  ]
}
```

For larger sample files with more options you can have a look a the [KrakenD Playground](/docs/v2.7/overview/playground/).
