---
lastmod: 2018-10-21
date: 2016-09-30
toc: true
linktitle: DNS SRV / Consul
title: Service Discovery with DNS SRV (e.g. Consul)
weight: 10
menu:
  documentation:
    parent: service-discovery
---
To integrate Consul as the Service Discovery or any other `DNS SRV` compatible systems you only need to set two keys:

- `"sd": "dns"`: To set service discovery = DNS SRV
- `"host": []`: The list of all the names providing the resolution

These keys need to be added in the `backend` section of your configuration. The `host` key can be skipped if there is another `host` key in the root level of the configuration.

For instance:

    ...
    "backend": [
            {
              "url_pattern": "/foo",
              "sd": "dns",
              "host": [
                "api-catalog.service.consul.srv"
              ]
            }
          ]
    ...
