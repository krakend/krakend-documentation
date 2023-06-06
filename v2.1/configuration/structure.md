---
lastmod: 2021-05-02
old_version: true
date: 2018-09-21
linktitle: The configuration file
menu:
  community_v2.1:
    parent: "010 Configuration file(s)"
title: Understanding the configuration file
weight: 10
---
All KrakenD behavior depends on its configuration file(s). Although the configuration [supports formats other than JSON](/docs/v2.1/configuration/supported-formats/) and it can be described [using multiple files](/docs/v2.1/configuration/flexible-config/), you'll find it referenced through all this documentation and for simplicity as the `krakend.json`. Being familiar with its structure it's essential.

## Configuration file structure
There are a large number of options you can put in this file. Let's focus now only on the main structure:
```json
{
    "$schema": "https://www.krakend.io/schema/v2.1/krakend.json",
    "version": 3,
    "endpoints": [],
    "extra_config": {}
}
```


- `$schema`: Optional, which JSON schema is used to validate your configuration. Is used by ` krakend check --lint`
- `version` (*mandatory*): The version of the KrakenD file format.
  - Version `3`: Current version (since `v2.0`)
  - Version `2`: Deprecated in 2022, for versions between `v0.4` and `v1.4.1`
  - Version `1`: Deprecated in 2016, for versions `v0.3.9` and older.
- `endpoints[]`: An array of endpoint objects offered by the gateway and all the associated backends and configurations. This is your API definition.
- `extra_config{}`: Components' configuration. Whatever is not a core functionality of the [Lura Project](https://luraproject.org) is declared in a unique **namespace** in the configuration, so that you can configure multiple elements without collisions.

### The `endpoints` structure
Inside the `endpoints`, you declare an array with every `endpoint` (the URL) the gateway offers to users. For each endpoint, you need to declare at least a `backend` (the data origin).

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

That's the basic structure of endpoints; for more information see [how to create endpoints](/docs/v2.1/endpoints/).

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
            ```
          }
        }
    }
```

For instance, the [extended logging component](/docs/v2.1/logging/) uses the **namespace** `telemetry/logging`:

```json
{
    "version": 3,
    "extra_config": {
        "telemetry/logging": {
          "level": "WARNING",
          "prefix": "[KRAKEND]",
          ```
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

For instance, you might want to set a rate limit between a user and KrakenD. And for that, you would place the `extra_config` inside the `endpoints` scope. Or you might want to limit the connections between KrakenD and your backends; then you would place the `extra_config` in the `backend` scope.

**You don't have to guess where to put the `extra_config`**. Each component has in the documentation what is the scope is built for.

### Example
The following code is an example defining two simultaneous rate limiting strategies: A limit of 5000 reqs/s for a specific endpoint, but yet, one of its backends accepts a maximum of 100 reqs/s. As you can imagine, when the backend limit is reached, the user will have partial responses.

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
        "backend":
        [{
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
        }]
    }]
}
```

Check [this larger sample file](https://github.com/krakendio/krakend-ce/blob/master/krakend.json) (distributed with KrakenD) where you can see an example on how to modify the application headers, configure the circuit breaker, or apply rate limits.
