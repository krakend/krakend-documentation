---
lastmod: 2021-01-28
date: 2018-09-21
linktitle: The configuration file
menu:
  documentation:
    parent: configuration
title: Understanding the configuration file
weight: 10
---
All KrakenD behavior depends on the `krakend.json` file ([other formats supported](/docs/configuration/supported-formats/)), so being familiar with the structure of the configuration file it's essential.

## Configuration file structure
There are a large number of options you can put in this file, let's focus now only on the structure:

    {
        "version": 2,
        "endpoints": [...]
        "extra_config": {...}
        ...
    }

- `version`: The KrakenD file format. Current version is `2`, use `1` only for old KrakenD releases (0.3.9 and below).
- `endpoints[]`: An array of endpoint objects offered by the gateway and all the associated backends and configurations.
- `extra_config{}`: Extra configuration associated with middlewares or components. For instance, you might want to enable logging or metrics, which are non-core and optional features of the API gateway.

### The `endpoints` structure
Inside the `endpoints`, you declare an array with every `endpoint` (the URL) the gateway offers. For each endpoint, you need to declare at least a `backend` - the place where the data is-.

It looks like this:

{{< highlight JSON >}}
{
    "endpoints": [
        {
          "endpoint": "/v1/foo-bar",
          "backend": [
            {
              "url_pattern": "/foo",
              "host": [
                "https://my.api.com"
              ]
            },
            {
              "url_pattern": "/bar",
              "host": [
                "https://my.api.com"
              ]
            }
          ]
        }
      ]
}
{{< /highlight >}}

This declares and endpoint `/v1/foo-bar` which is the result of merging the responses from `/foo` and `/bar`.

### The `extra_config` structure
When a component is registered, its associated configuration is taken from the `extra_config` (if any).

The `extra_config` can appear on different levels, and this depends entirely on every component. An `extra_config` in the root level of the file usually sets values at a service level, and this affects the gateway globally. On the other hand, some components seek the `extra_config` key inside an `endpoint` definition or a `backend` definition, as its functionality is specific to the backend or the endpoint behavior only. For instance, you might want to set a rate limit only to a specific endpoint or backend.

It's the responsibility of each external component to define a **namespace** that will be used as the key to retrieve the configuration. For instance, the [gologging middleware](https://github.com/devopsfaith/krakend-gologging) expects to find a key `github_com/devopsfaith/krakend-gologging`:
{{< highlight JSON >}}
{
    "version": 2,
    "extra_config": {
        "github_com/devopsfaith/krakend-gologging": {
          "level": "WARNING",
          "prefix": "[KRAKEND]",
          "syslog": false,
          "stdout": true
        }
    }
}
{{< /highlight >}}

As per the official KrakenD components, the namespaces use the library path as the key for the `extra_config` as this is considered a good practice. When the `extra_config` is in the root of the file (service level) the namespace does not use any dot (notice the `github_com`) to avoid problems with parsers, but when the extra_config is placed at `endpoint` level or even `backend` level, the dots are present.

The following code is an example defining two simultaneous rate limiting strategies: A limit of 5000 reqs/s for a specific endpoint, but yet, one of its backends accepts a maximum of 100 reqs/s.

{{< highlight JSON "hl_lines=8 19" >}}
{
    "version": 2,
    "endpoints": [
    {
        "endpoint": "/limited-to-5000-per-second",
        "extra_config": {
            "github.com/devopsfaith/krakend-ratelimit/juju/router": {
                "maxRate": 5000
            }
        },
        "backend":
        [{
            "host": [
                "http://slow.backend.com/"
            ],
            "url_pattern": "/slow/endpoint",
            "extra_config": {
                "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
                    "maxRate": 100,
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
        ...
    }
}
{{< /highlight >}}
### Example file

Check [this larger sample file](https://github.com/devopsfaith/krakend-ce/blob/master/krakend.json) (distributed with KrakenD) where you can see an example on how to modify the application headers, configure the circuit breaker or apply rate limits.

### Advanced tips for shorter configurations
To keep shorter configuration files, and easier to read, have a look at the best practices to do [housekeeping of your configuration files](/blog/housekeeping-configuration-file/)
