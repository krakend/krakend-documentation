---
lastmod: 2018-10-21
date: 2016-09-30
toc: true
linktitle: Service Discovery overview
title: Service Discovery overview
weight: -1000
menu:
  documentation:
    parent: service-discovery
---

Service discovery enables clients to detect and locate services on your enterprise network automatically. Instead of defining a static list of IPs or hostnames pointing to your backends, you can use a service discovery provider and let KrakenD interact with it to get the hosts dynamically.

# DNS SRV
A common Service Discovery integration for KrakenD is to use `DNS SRV`, similarly the way many other systems work such as Kubernetes.

The most typical setup is KrakenD + [Consul](https://www.consul.io/) (see [standard lookup](https://www.consul.io/docs/agent/dns.html#standard-lookup)).

# etcd
KrakenD can watch the values of the keys in an etcd installation and reconfigure itself when they change.

The integration with etcd allows you to set up your distributed key-value store and configure details such as the timeouts, keep-alive, and certificates.





