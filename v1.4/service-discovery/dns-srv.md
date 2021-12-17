---
lastmod: 2019-02-13
old_version: true
date: 2016-09-30no
notoc: true
linktitle: DNS SRV
title: SD with DNS SRV (e.g., Consul, k8s)
weight: 10
menu:
  community_v1.4:
    parent: "050 Backends Configuration"
---
The `DNS SRV` is a market standard used by systems such as **Kubernetes, Mesos, Haproxy, Nginx plus, AWS ECS, Linkerd**, and more.

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
              ],
              "disable_host_sanitize": true
            }
          ]
    ...
