---
lastmod: 2018-07-12
date: 2016-07-01
linktitle: Configuration file
menu:
  main:
    parent: getting started
title: KrakenD configuration - The krakend.json file
weight: 40
aliases:
- /docs/configuration/configuration-start/
- /docs/configuration/cluster
- /docs/configuration/standalone
---

All KrakenD configuration lives inside a unique file. We will refer to it as the `krakend.json` file
or the `/etc/krakend/krakend.json` but it will have the name and live in the location that you decide.

The `krakend.json` file defines endpoints, business logic, service limits, security configuration,
SSL certificates and any other piece of configuration needed by KrakenD or your own middlewares.

# Generating a configuration file
The easiest way to write your first configuration file is by simply using the [KrakenDesigner](http://designer.krakend.io/).
This is a simple javascript application that helps you understand
and set the different values for all the options without learning from scratch the value names for
every attribute. When you are done it allows you to download the settings.
You can resume the edition of any existing configuration file by dragging the file in the designer.

In the [designer](http://designer.krakend.io/) **no data is uploaded to the server ever** and everything stays only in your browser.

Although the KrakenD framework is ready to parse formats like `yaml` or `toml` the KrakenDesigner works only
with `JSON` files. We recommend sticking to JSON until you are very familiar with the product.


<a class="btn btn-primary btn-circle" href="http://designer.krakend.io/">Generate configuration now</a>

# Validating the syntax of the configuration file
Let's say you are modifying by hand the file `~/my-krakend.json`. When you are set, you can check the syntax of the file by running:

    krakend --config ~/my-krakend.json check

By adding the flag `--debug` you will be able to see the full output with the interpretation of the file:

    krakend --config ~/my-krakend.json --debug

Of course you can try to start the service directly as this will be done anyway. When the syntax is correct you'll see the message

    Syntax OK!

The configuration file contains a lot of different options that are not explained in this section, the best way to get familiar with them is using the KrakenDesigner.

# Configuration file structure
The configuration file contains all the options defining how the gateway behaves. There are a large number of options you can put in this file, but in this section we will focus only in the structure:

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
Inside the `endpoints` you will basically declare an array with every `endpoint` definition (the URL) the gateway will answer to. For each endpoint you need to declare at least a `backend`, the place were your API with the data lives. It looks like this:

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
When a middleware is registered, its configuration is taken from the `extra_config`.

Every middleware will look up for a key containing its own go library **namespace**. For instance, let's say that we want to enable the [golooging middleware](https://github.com/devopsfaith/krakend-ratelimit), then we only need to add the namespace and the associated configuration. Every middleware has its own configuration, so the content inside the namespace depends on every middelware:
{{< highlight JSON >}}
{
	"extra_config": {
	    "github_com/devopsfaith/krakend-gologging": {
	      "level": "WARNING",
	      "prefix": "[KRAKEND]",
	      "syslog": false,
	      "stdout": true
	    }
}
{{< /highlight >}}

When the `extra_config` is in the root of the file (service level) the namespace does not use any dot (notice the `github_com`) but the extra_config can also be placed at `endpoint` level or even `backend` level. This is an example defining a global limit of 5000 reqs/s for the entire Gateway, but a specific endpoint accepting only 10 reqs/s.
{{< highlight JSON "hl_lines=5 14" >}}
	{
	  "version": 2,
	  "extra_config": {
	    "github_com/devopsfaith/krakend-ratelimit/juju/router": {
	      "maxRate": 5000,
	      "clientMaxRate": 0
	    }
	  },
	  "endpoints": [
	    {
	      "endpoint": "/limited-to-10-per-second",
	      "extra_config": {
	        "github.com/devopsfaith/krakend-ratelimit/juju/router": {
	          "maxRate": 10
	        }
	      },
	      "backend": [...]
	    },
	    ...
	  ],
	  ...
	}
{{< /highlight >}}
## Example file

Check [this larger sample file](https://github.com/devopsfaith/krakend-ce/blob/master/krakend.json) (distributed with KrakenD) where you will see an example on how to modify the application headers, configure the circuit breaker or apply rate limits.

# Flexible configuration
There might be times when you want to split the content of a large configuration file, or use placeholders instead of hardcoding values in the configuration file because you have multiple environments, or any other reason.

Instead of working with the final configuration file, the package **[krakend-flexibleconfig](https://github.com/devopsfaith/krakend-flexibleconfig)** allows you to use a template that will generate the final configuration file.

This is an example of a flexible configuration template:

{{< highlight go "hl_lines=3 10 19 24 32-33 39 44" >}}
{
    "version": 2,
    "port": {{ .Port }},
    "endpoints": [
        {
            "endpoint": "/combination/{id}",
            "backend": [
                {
                    "host": [
                        "{{ .Jsonplaceholder }}"
                    ],
                    "url_pattern": "/posts?userId={id}",
                    "is_collection": true,
                    "mapping": {
                        "collection": "posts"
                    },
                    "disable_host_sanitize": true,
                    "extra_config": {
                        "namespace1": {{ marshal .Namespace1 }}
                    }
                },
                {
                    "host": [
                        "{{ .Jsonplaceholder }}"
                    ],
                    "url_pattern": "/users/{id}",
                    "mapping": {
                        "email": "personal_email"
                    },
                    "disable_host_sanitize": true,
                    "extra_config": {
                        "namespace1": {{ marshal .Namespace1 }},
                        "namespace2": {{ marshal .Namespace2 }}
                    }
                }
            ],
            "extra_config": {
                "namespace3": { "foo": "bar" },
                "namespace2": {{ marshal .Namespace2 }}
            }
        }
    ],
    "extra_config": {
        "namespace2": {{ marshal .Namespace2 }}
    }
}
{{< /highlight >}}