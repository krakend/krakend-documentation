---
lastmod: 2019-02-13
old_version: true
date: 2016-09-30
toc: true
linktitle: Service Discovery overview
title: Service Discovery overview
weight: -1000
menu:
  community_v1.4:
    parent: "050 Backends Configuration"
---

Service discovery enables clients to detect and locate services on your enterprise network automatically. Instead of defining a static list of IPs or hostnames pointing to your backends, you can use a service discovery provider and let KrakenD interact with it to get the hosts dynamically.

## Static resolution
The `static` resolution is the default service discovery choice. It uses a list of hosts declared in the configuration file and KrakenD must be able to reach them directly by hostname, DNS or IP. All the necessary connections are load balanced between all the servers in the list.

Using `"sd": "static"` in the configuration file is optional.

```
"backend": [
	{
		"url_pattern": "/some-url",
		"sd": "static",
		"host": [
			"http://my-service-01.api.com:9000",
			"http://my-service-02.api.com:9000"
		]
	}
]
```

## DNS SRV
A typical Service Discovery integration for KrakenD is to use `DNS SRV`. It is a market standard used by many other systems work such as **Kubernetes, Mesos, Haproxy, Nginx plus, AWS ECS, Linkerd**, and more.

The most typical setup is KrakenD + [Consul](https://www.consul.io/) (see [standard lookup](https://www.consul.io/docs/agent/dns.html#standard-lookup)).

See [DNS SRV service discovery](/docs/v1.4/service-discovery/dns-srv/)

## etcd
KrakenD can watch the values of the keys in an etcd installation and reconfigure itself when they change.

The integration with etcd allows you to set up your distributed key-value store and configure details such as the timeouts, keep-alive, and certificates.

See [etcd service discovery](/docs/v1.4/service-discovery/etcd/)

## Eureka
There is support for the Netflix Eureka client. The package is an open source contribution via [Schibsted](https://www.schibsted.com/) engineers.

See [Eureka service discovery](/docs/v1.4/service-discovery/eureka/)
