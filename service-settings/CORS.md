---
lastmod: 2020-07-10
date: 2020-07-10
linktitle: CORS
title: Enabling Cross Origin Resource Sharing (CORS)
weight: 20
notoc: true
meta:
  since: 0.6
  source: https://github.com/devopsfaith/krakend-cors
  namespace:
  - security/cors
  log_prefix:
  - "[SERVICE: Gin][CORS]"
  scope:
  - service

menu:
  community_current:
    parent: "030 Service Settings"
---
When KrakenD endpoints are consumed from a browser, you might need to enable the **Cross-Origin Resource Sharing (CORS)** module as browsers restrict cross-origin HTTP requests initiated from scripts.

When the Cross-Origin Resource Sharing (CORS) configuration is enabled, KrakenD uses additional HTTP headers to tell browsers that they can **use resources from a different origin** (domain, protocol, or port). For instance, you will need this configuration if your web page is hosted at https://www.domain.com and the Javascript references the KrakenD API at https://api.domain.com.

## Configuration
CORS configuration lives in the root of the file, as it's a service component. Add the namespace `security/cors` under the global `extra_config`, as follows:

{{< highlight json >}}
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
      "allow_headers": [
        "Accept-Language"
      ],
      "allow_credentials": false,
      "debug": false
    }
  }
{{< /highlight >}}
The configuration options of this component are as follows:

- `allow_methods` *(list)*: The array of all HTTP methods accepted, in uppercase.
- `allow_origins` *(list)*: An array with all the origins allowed, examples of values are `https://example.com`, or `*` (any origin).
- `allow_headers` *(list)*: An array with the headers allowed. Missing headers in this list won't be accepted.
- `expose_headers` *(list)*: Headers that are safe to expose to the API of a CORS API specification
- `max_age` *(string)*: For how long the response can be cached. The value needs to specify units. Valid time units are: `ns`, `us`, (or `µs`), `ms`, `s`, `m`, `h` E.g., `12h` for 12 hours.
- `allow_credentials` *(boolean)*: When requests can include user credentials like cookies, HTTP authentication or client side SSL certificates
- `debug`: *(boolean)*: Show debugging information in the logger, **to be used only during development** (defaults to `false`)

{{< note title="Allow credentials and wildcards" >}}
According to the CORS specification, you are not allowed to use wildcards and credentials at the same time. If you need to do this, [check this workaround](https://github.com/devopsfaith/krakend-cors/issues/9){{< /note >}}

## Debugging configuration
The following configuration might help you debugging your CORS configuration. Check the inline `@comments`:

{{< highlight json >}}
{
  "endpoints":[
        {
            "@comment": "this will fail due to double CORS validation",
            "endpoint":"/cors/no-op",
            "headers_to_pass":["*"],
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
            "headers_to_pass":["*"],
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
{{< /highlight >}}