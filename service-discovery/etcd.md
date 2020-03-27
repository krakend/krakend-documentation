---
lastmod: 2019-02-13
date: 2016-09-30
toc: true
linktitle: etcd
title: Service Discovery with etcd
weight: 20
source: https://github.com/devopsfaith/krakend-etcd
menu:
  documentation:
    parent: service-discovery
---

The etcd Service Discovery integration allows you to perform the host resolution using your existing etcd setup.

The integration etcd is controlled by the [krakend-etcd](https://github.com/devopsfaith/krakend-etcd) component and adds client and subscriber capabilities for etcd.


## Enabling etcd
To enable the integration add in the **root** of your configuration file the necessary settings:

{{< highlight json "hl_lines=3-15" >}}
{
  "version": 2,
  "extra_config": {
    "github_com/devopsfaith/krakend-etcd": {
      "machines": [
        "https://192.168.1.100:4001",
        "https://192.168.1.101:4001"
      ],
      "dial_timeout": "5s",
      "dial_keepalive": "30s",
      "header_timeout": "1s",
      "cert": "/path/to/cert",
      "key": "/path/to/cert-private-key",
      "cacert": "/path/to/CA-cert"
    }
  },
  ...
{{< /highlight >}}}

The only mandatory key inside the `github_com/devopsfaith/krakend-etcd` namespace is `machines`, so the integration knows where etcd lives. The rest of the keys in the configuration are optional.

The timeouts and keepalive will take the shown values if the keys are missing, and the certificates (`cert`, `key`, and `cacert`) are not needed if you use plain connections.

Now you are able to use `etcd` with your backends.

## Using etcd to resolve backends
Once the integration has been enabled, add the following keys in the `backend` section of your configuration. The `host` key can be skipped if there is another `host` key in the root level of the configuration.

- `"sd": "etcd"`: To set etcd as the service discovery
- `"host": []`: The list of all the services you want to resolve

For instance, set the service discovery for an individual backend:

    ...
    "backend": [
            {
              "url_pattern": "/foo",
              "sd": "etcd",
              "host": [
                "api-catalog"
              ],
              "disable_host_sanitize": true
            }
          ]
    ...
