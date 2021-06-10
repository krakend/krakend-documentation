---
lastmod: 2021-02-16
date: 2016-10-28
linktitle: Check
title: Commands - check
description: KrakenD Check Command
weight: 10
menu:
  community_v1.3:
    parent: "020 Command Line"
---

The `krakend check` command validates the passed configuration. Since KrakenD does not implement a strict parsing, typos
 in the config file could be shadowed. In order to validate your config completely, it is recommended to use the `--debug` flag.

{{< terminal title="Usage of KrakenD check" >}}
./krakend check -h
                                                                        
`7MMF' `YMM'                  `7MM                         `7MM"""Yb.   
  MM   .M'                      MM                           MM    `Yb. 
  MM .d"     `7Mb,od8 ,6"Yb.    MM  ,MP'.gP"Ya `7MMpMMMb.    MM     `Mb 
  MMMMM.       MM' "'8)   MM    MM ;Y  ,M'   Yb  MM    MM    MM      MM 
  MM  VMA      MM     ,pm9MM    MM;Mm  8M""""""  MM    MM    MM     ,MP 
  MM   `MM.    MM    8M   MM    MM `Mb.YM.    ,  MM    MM    MM    ,dP' 
.JMML.   MMb..JMML.  `Moo9^Yo..JMML. YA.`Mbmmd'.JMML  JMML..JMMmmmdP'   
_______________________________________________________________________
                                                                  
Version: {{< version >}}

Validates that the active configuration file has a valid syntax to run the service.
Change the configuration file by using the --config flag

Usage:
  krakend check [flags]

Aliases:
  check, validate

Examples:
krakend check -d -c config.json

Flags:
  -h, --help              help for check
  -t, --test-gin-routes   Test the endpoint patterns against a real gin router on selected port

Global Flags:
  -c, --config string   Path to the configuration filename
  -d, --debug           Enable the debug
{{< /terminal >}}

Passing a path to the config file is required

{{< terminal >}}
krakend check
Please, provide the path to your config file
{{< /terminal >}}

## Example

We will use this configuration for the demo

	{
	    "version": 2,
	    "name": "My lovely gateway",
	    "port": 8080,
	    "cache_ttl": "3600s",
	    "timeout": "3s",
	    "extra_config": {
	      "github_com/devopsfaith/krakend-gologging": {
	        "level":  "ERROR",
	        "prefix": "[KRAKEND]",
	        "syslog": false,
	        "stdout": true
	      }
	    },
	    "endpoints": [
	        {
	            "endpoint": "/supu",
	            "method": "GET",
	            "backend": [
	                {
	                    "host": [
	                        "http://127.0.0.1:8080"
	                    ],
	                    "url_pattern": "/__debug/supu",
	                    "extra_config": {
	                        "github.com/devopsfaith/krakend-martian": {
	                            "fifo.Group": {
	                                "scope": ["request", "response"],
	                                "aggregateErrors": true,
	                                "modifiers": [
	                                {
	                                    "header.Modifier": {
	                                        "scope": ["request", "response"],
	                                        "name" : "X-Martian",
	                                        "value" : "ouh yeah!"
	                                    }
	                                },
	                                {
	                                    "body.Modifier": {
	                                        "scope": ["request"],
	                                        "contentType" : "application/json",
	                                        "body" : "eyJtc2ciOiJ5b3Ugcm9jayEifQ=="
	                                    }
	                                },
	                                {
	                                    "header.RegexFilter": {
	                                        "scope": ["request"],
	                                        "header" : "X-Neptunian",
	                                        "regex" : "no!",
	                                        "modifier": {
	                                            "header.Modifier": {
	                                                "scope": ["request"],
	                                                "name" : "X-Martian-New",
	                                                "value" : "some value"
	                                            }
	                                        }
	                                    }
	                                }
	                                ]
	                            }
	                        },
	                        "github.com/devopsfaith/krakend-circuitbreaker/gobreaker": {
	                            "interval": 60,
	                            "timeout": 10,
	                            "maxErrors": 1
	                        }
	                    }
	                }
	            ]
	        },
	        {
	            "endpoint": "/github/{user}",
	            "method": "GET",
	            "backend": [
	                {
	                    "host": [
	                        "https://api.github.com"
	                    ],
	                    "url_pattern": "/",
	                    "allow": [
	                        "authorizations_url",
	                        "code_search_url"
	                    ],
	                    "disable_host_sanitize": true,
	                    "extra_config": {
	                        "github.com/devopsfaith/krakend-martian": {
	                            "fifo.Group": {
	                                "scope": ["request", "response"],
	                                "aggregateErrors": true,
	                                "modifiers": [
	                                {
	                                    "header.Modifier": {
	                                        "scope": ["request", "response"],
	                                        "name" : "X-Martian",
	                                        "value" : "ouh yeah!"
	                                    }
	                                },
	                                {
	                                    "body.Modifier": {
	                                        "scope": ["request"],
	                                        "contentType" : "application/json",
	                                        "body" : "eyJtc2ciOiJ5b3Ugcm9jayEifQ=="
	                                    }
	                                },
	                                {
	                                    "header.RegexFilter": {
	                                        "scope": ["request"],
	                                        "header" : "X-Neptunian",
	                                        "regex" : "no!",
	                                        "modifier": {
	                                            "header.Modifier": {
	                                                "scope": ["request"],
	                                                "name" : "X-Martian-New",
	                                                "value" : "some value"
	                                            }
	                                        }
	                                    }
	                                }
	                                ]
	                            }
	                        },
	                        "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
	                            "maxRate": 2,
	                            "capacity": 2
	                        },
	                        "github.com/devopsfaith/krakend-circuitbreaker/gobreaker": {
	                            "interval": 60,
	                            "timeout": 10,
	                            "maxErrors": 1
	                        }
	                    }
	                }
	            ]
	        },
	        {
	            "endpoint": "/show/{id}",
	            "backend": [
	                {
	                    "host": [
	                        "http://showrss.info/"
	                    ],
	                    "url_pattern": "/user/schedule/{id}.rss",
	                    "encoding": "rss",
	                    "group": "schedule",
	                    "allow": ["items", "title"],
	                    "extra_config": {
	                        "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
	                            "maxRate": 1,
	                            "capacity": 1
	                        },
	                        "github.com/devopsfaith/krakend-circuitbreaker/gobreaker": {
	                            "interval": 60,
	                            "timeout": 10,
	                            "maxErrors": 1
	                        }
	                    }
	                },
	                {
	                    "host": [
	                        "http://showrss.info/"
	                    ],
	                    "url_pattern": "/user/{id}.rss",
	                    "encoding": "rss",
	                    "group": "available",
	                    "allow": ["items", "title"],
	                    "extra_config": {
	                        "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
	                            "maxRate": 2,
	                            "capacity": 2
	                        },
	                        "github.com/devopsfaith/krakend-circuitbreaker/gobreaker": {
	                            "interval": 60,
	                            "timeout": 10,
	                            "maxErrors": 1
	                        }
	                    }
	                }
	            ],
	            "extra_config": {
	                "github.com/devopsfaith/krakend-ratelimit/juju/router": {
	                    "maxRate": 50,
	                    "clientMaxRate": 5,
	                    "strategy": "ip"
	                }
	            }
	        }
	    ]
	}


## Debug disabled
{{< terminal title="Checking the configuration without the debug flag" >}}
krakend check --config krakend.json
Parsing configuration file: krakend.json
Syntax OK!
{{< /terminal >}}

## Debug enabled
{{< terminal title="Checking the configuration with the debug flag" >}}
krakend check -c krakend.json -d
Parsing configuration file: krakend.json
Parsed configuration: CacheTTL: 1h0m0s, Port: 8080
Hosts: []
Extra (1):
	github_com/devopsfaith/krakend-gologging: map[level:ERROR syslog:false stdout:true prefix:[KRAKEND]]
Endpoints (3):
	Endpoint: /supu, Method: GET, CacheTTL: 1h0m0s, Concurrent: 1, QueryString: []
	Extra (0):
	Backends (1):
		URL: /__debug/supu, Method: GET
			Timeout: 3s, Target: , Mapping: map[], BL: [], WL: [], Group:
			Hosts: [http://127.0.0.1:8080]
			Extra (2):
				github.com/devopsfaith/krakend-martian: map[fifo.Group:map[scope:[request response] aggregateErrors:true modifiers:[map[header.Modifier:map[value:ouh yeah! scope:[request response] name:X-Martian]] map[body.Modifier:map[scope:[request] contentType:application/json body:eyJtc2ciOiJ5b3Ugcm9jayEifQ==]] map[header.RegexFilter:map[header:X-Neptunian regex:no! modifier:map[header.Modifier:map[scope:[request] name:X-Martian-New value:some value]] scope:[request]]]]]]
				github.com/devopsfaith/krakend-circuitbreaker/gobreaker: map[timeout:10 maxErrors:1 interval:60]
	Endpoint: /github/:user, Method: GET, CacheTTL: 1h0m0s, Concurrent: 1, QueryString: []
	Extra (0):
	Backends (1):
		URL: /, Method: GET
			Timeout: 3s, Target: , Mapping: map[], BL: [], WL: [authorizations_url code_search_url], Group:
			Hosts: [https://api.github.com]
			Extra (3):
				github.com/devopsfaith/krakend-martian: map[fifo.Group:map[modifiers:[map[header.Modifier:map[name:X-Martian value:ouh yeah! scope:[request response]]] map[body.Modifier:map[scope:[request] contentType:application/json body:eyJtc2ciOiJ5b3Ugcm9jayEifQ==]] map[header.RegexFilter:map[scope:[request] header:X-Neptunian regex:no! modifier:map[header.Modifier:map[scope:[request] name:X-Martian-New value:some value]]]]] scope:[request response] aggregateErrors:true]]
				github.com/devopsfaith/krakend-ratelimit/juju/proxy: map[maxRate:2 capacity:2]
				github.com/devopsfaith/krakend-circuitbreaker/gobreaker: map[interval:60 timeout:10 maxErrors:1]
	Endpoint: /show/:id, Method: GET, CacheTTL: 1h0m0s, Concurrent: 1, QueryString: []
	Extra (1):
		github.com/devopsfaith/krakend-ratelimit/juju/router: map[maxRate:50 clientMaxRate:5 strategy:ip]
	Backends (2):
		URL: /user/schedule/{{.Id}}.rss, Method: GET
			Timeout: 3s, Target: , Mapping: map[], BL: [], WL: [items title], Group: schedule
			Hosts: [http://showrss.info]
			Extra (2):
				github.com/devopsfaith/krakend-circuitbreaker/gobreaker: map[interval:60 timeout:10 maxErrors:1]
				github.com/devopsfaith/krakend-ratelimit/juju/proxy: map[maxRate:1 capacity:1]
		URL: /user/{{.Id}}.rss, Method: GET
			Timeout: 3s, Target: , Mapping: map[], BL: [], WL: [items title], Group: available
			Hosts: [http://showrss.info]
			Extra (2):
				github.com/devopsfaith/krakend-ratelimit/juju/proxy: map[maxRate:2 capacity:2]
				github.com/devopsfaith/krakend-circuitbreaker/gobreaker: map[interval:60 timeout:10 maxErrors:1]
Syntax OK!
{{< /terminal >}}

## Check conflicting routes
Even the configuration is valid from a syntax perspective, you can have a failing KrakenD once the service starts. To avoid this situation, the `-t` flag will actually start a KrakenD instance and **test the routes**.

When automating the checks of the configuration, make sure to add the `-t` flag:

```
krakend check -t -d -c config.json
```

Make sure that the port of KrakenD is not allocated in your pipeline. You can always change it with environment vars.