---
lastmod: 2018-09-27
date: 2018-09-21
linktitle: Understanding the file
menu:
  documentation:
    parent: configuration
title: Understanding the configuration file
weight: 10
---

# Configuration file structure
The configuration file contains all the options defining how the gateway behaves. There are a large number of options you can put in this file, but in this section, we will focus only on the structure:

	{
	    "version": 2,
	    "endpoints": [...]
	    "extra_config": {...}
	    ...
	}

- `version`: Tells KrakenD what syntax version needs to use. Supported versions are 1 (krakend 0.3.9 and below) or 2 (current).
- `endpoints`: Every single endpoint offered by the gateway and all the associated backends will live here
- `extra_config`: In case there is any extra middleware (non-core) needing configuration this will be here. For instance, you might want to enable logging, metrics, throttling, etc. which are middleware.

## The `endpoints` structure
Inside the `endpoints`, you will basically declare an array with every `endpoint` definition (the URL) the gateway will answer to. For each endpoint, you need to declare at least a `backend`, the place were your API with the data lives. It looks like this:

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

## The `extra_config` structure
When a component is registered, its configuration is taken from the `extra_config` (if any). The `extra_config` can be declared in the root level of the file, but many components will look for the `extra_config` key inside an `endpoint` definition or a `backend` definition.

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

As per the official KrakenD components, the namespaces use the library path as the key for the `extra_config` as this is considered a good practice. When the `extra_config` is in the root of the file (service level) the namespace does not use any dot (notice the `github_com`) to avoid problems with parsers, but when the extra_config is placed at `endpoint` level or even `backend` level the dots are present.

The following code is an example defining two simultaneous rate limiting strategies: A limit of 5000 reqs/s for a specific endpoint, but yet, one of its backends will accept a maximum of 100 reqs/s.

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
## Example file

Check [this larger sample file](https://github.com/devopsfaith/krakend-ce/blob/master/krakend.json) (distributed with KrakenD) where you will see an example on how to modify the application headers, configure the circuit breaker or apply rate limits.
