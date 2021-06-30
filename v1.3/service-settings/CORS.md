---
lastmod: 2020-07-10
old_version: true
date: 2020-07-10
linktitle: CORS
title: Enabling Cross Origin Resource Sharing (CORS)
weight: 20
notoc: true
meta:
  since: 0.6
  source: https://github.com/devopsfaith/krakend-cors
  namespace:
  - github_com/devopsfaith/krakend-cors
  scope:
  - service

menu:
  community_v1.3:
    parent: "030 Service Settings"
---
When KrakenD endpoints are consumed from a browser, you might need to enable the **Cross-Origin Resource Sharing (CORS)** module as browsers restrict cross-origin HTTP requests initiated from scripts.

When the Cross-Origin Resource Sharing (CORS) configuration is enabled, KrakenD uses additional HTTP headers to tell browsers that they can **use resources from a different origin** (domain, protocol, or port). For instance, you will need this configuration if your web page is hosted at https://domain-a.com and the Javascript references the KrakenD API at https://domain-b.com.

## Configuration
CORS configuration lives in the root of the file, as it's a service component. Add the namespace `github_com/devopsfaith/krakend-cors` under the global `extra_config`, as follows:

    {
      "version": 2,
      "extra_config": {
        "github_com/devopsfaith/krakend-cors": {
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
          "max_age": "12h",
          "allow_headers": [
            "Accept-Language"
          ],
          "allow_credentials": false,
          "debug": false
        }
      }

The configuration options of this component are as follows:

- `allow_methods` *(list)*: The array of all HTTP methods accepted, in uppercase.  
- `allow_origins` *(list)*: An array with all the origins allowed, examples of values are `https://example.com`, or `*` (any origin).
- `expose_headers` *(list)*: Headers that are safe to expose to the API of a CORS API specification
- `max_age` *(string)*: For how long the response can be cached. The value needs to specify units. Valid time units are: `ns`, `us`, (or `µs`), `ms`, `s`, `m`, `h` E.g., `12h` for 12 hours.
- `allow_credentials` *(boolean)*: When requests can include user credentials like cookies, HTTP authentication or client side SSL certificates
- `debug`: *(boolean)*: Show debugging information in the logger, **to be used only during development** (defaults to `false`)

{{< note title="Allow credentials and wildcards" >}}
According to the CORS specification, you are not allowed to use wildcards and credentials at the same time. If you need to do this, [check this workaround](https://github.com/devopsfaith/krakend-cors/issues/9){{< /note >}}
