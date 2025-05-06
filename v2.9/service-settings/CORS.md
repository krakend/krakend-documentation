---
lastmod: 2023-06-27
old_version: true
date: 2020-07-10
linktitle: CORS
title: Cross-Origin Resource Sharing (CORS) Configuration
description: Configure Cross-Origin Resource Sharing (CORS) in KrakenD API Gateway to enable secure communication between different domains and APIs.
weight: 20
notoc: true
meta:
  since: 0.6
  source: https://github.com/krakend/krakend-cors
  namespace:
  - security/cors
  log_prefix:
  - "[SERVICE: Gin][CORS]"
  scope:
  - service

menu:
  community_v2.9:
    parent: "070 Security"
---
When KrakenD endpoints are consumed from a browser, you should enable the **Cross-Origin Resource Sharing (CORS)** module, as browsers restrict cross-origin HTTP requests initiated from scripts.

When the Cross-Origin Resource Sharing (CORS) configuration is enabled, KrakenD uses additional HTTP headers to tell browsers that they can **use resources from a different origin** (domain, protocol, or port). For instance, you will need this configuration if your web page is hosted at https://www.domain.com and the Javascript references the KrakenD API at https://api.domain.com.

## Configuration
CORS configuration lives in the root of the file, as it's a service component. Add the namespace `security/cors` under the global `extra_config` as follows:

```json
{
  "version": 3,
  "extra_config": {
    "security/cors": {
      "allow_origins": [
        "*"
      ],
      "allow_methods": [
        "GET",
        "HEAD",
        "POST"
      ],
      "expose_headers": [
        "Content-Length",
        "Content-Type"
      ],
      "allow_headers": [
        "Accept-Language"
      ],
      "max_age": "12h",
      "allow_credentials": false,
      "debug": false
    }
  }
}
```
The configuration options of this component are as follows:

{{< schema version="v2.9" data="security/cors.json" >}}

{{< note title="Allow credentials and wildcards" >}}
According to the CORS specification, you are not allowed to use wildcards and credentials at the same time. If you need to do this, [check this workaround](https://github.com/krakend/krakend-cors/issues/9){{< /note >}}

## Debugging configuration
The following configuration might help you debug your CORS configuration. Check the inline `@comments`:

```json
{
  "endpoints":[
        {
            "@comment": "this will fail due to double CORS validation",
            "endpoint":"/cors/no-op",
            "input_headers":["*"],
            "output_encoding": "no-op",
            "backend":[
                {
                    "url_pattern": "/__debug/cors",
                    "host": ["http://localhost:8080"],
                    "encoding": "no-op"
                }
            ]
        },
        {
            "@comment": "this won't fail because CORS preflight headers are removed from the request to the backend",
            "endpoint":"/cors/no-op/martian",
            "input_headers":["*"],
            "output_encoding": "no-op",
            "backend":[
                {
                    "url_pattern": "/__debug/cors/martian",
                    "host": ["http://localhost:8080"],
                    "encoding": "no-op",
                    "extra_config":{
                      "modifier/martian": {
                          "fifo.Group": {
                              "scope": ["request", "response"],
                              "aggregateErrors": true,
                              "modifiers": [
                                  {
                                    "header.Blacklist": {
                                      "scope": ["request"],
                                      "names": [
                                        "Access-Control-Request-Method",
                                        "Sec-Fetch-Dest",
                                        "Sec-Fetch-Mode",
                                        "Sec-Fetch-Site",
                                        "Origin"
                                      ]
                                    }
                                  }
                              ]
                          }
                      }
                    }
                }
            ]
        },
        {
            "@comment": "this won't fail because no headers are added to the request to the backend",
            "endpoint":"/cors/no-op/no-headers",
            "output_encoding": "no-op",
            "backend":[
                {
                    "url_pattern": "/__debug/cors/no-headers",
                    "host": ["http://localhost:8080"],
                    "encoding": "no-op"
                }
            ]
        }
]}
```

## Adding the OPTIONS method
When working in a SPA, you will usually receive `OPTIONS` calls to KrakenD; although this configuration is not related to CORS, it usually goes in hand.

To support `OPTIONS` in your endpoints, you only need to add the [flag `auto_options`](/docs/v2.9/service-settings/router-options/#auto_options) as follows:

```json
{
  "version": 3,
  "extra_config": {
    "router": {
       "auto_options": true
    },
    "security/cors": {
      "@comment": "...CORS configuration inside this block..."
    }
  }
}
```
{{< schema version="v2.9" data="router.json" filter="auto_options" >}}
