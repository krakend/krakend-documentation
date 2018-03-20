---
lastmod: 2016-09-17
date: 2016-07-01
linktitle: Configuration
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

## Configuration file example
This is simplified example of a configuration file:

    {
	  "version": 2,
	  "extra_config": {
	    "github_com/devopsfaith/krakend-gologging": {
	      "level": "ERROR",
	      "prefix": "[KRAKEND]",
	      "syslog": false,
	      "stdout": true
	    }
	  },
	  "timeout": "3000ms",
	  "cache_ttl": "300s",
	  "sd_providers": {
	    "hosts": [
	      {
	        "sd": "static",
	        "host": "http://api.company.local:9000"
	      }
	    ]
	  },
	  "endpoints": [
	    {
	      "endpoint": "/products",
	      "method": "GET",
	      "concurrent_calls": 1,
	      "extra_config": {
	        "github_com/devopsfaith/krakend-httpsecure": {
	          "disable": true,
	          "allowed_hosts": [],
	          "ssl_proxy_headers": {}
	        },
	        "github.com/devopsfaith/krakend-ratelimit/juju/router": {
	          "clientMaxRate": 0
	        }
	      },
	      "backend": [
	        {
	          "url_pattern": "/products-list",
	          "extra_config": {
	            "github.com/devopsfaith/krakend-oauth2-clientcredentials": {
	              "is_disabled": true,
	              "endpoint_params": {}
	            }
	          },
	          "encoding": "json"
	        }
	      ]
	    }
	  ]
	}

Check [this larger sample file](https://github.com/devopsfaith/krakend-ce/blob/master/krakend.json) (distributed with KrakenD) where you will see an example on how to modify the application headers, configure the circuit breaker or apply rate limits.