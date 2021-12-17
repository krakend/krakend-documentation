---
lastmod: 2021-12-16
date: 2016-09-30
aliases: ["/service-discovery/dns-srv/", "/docs/service-discovery/overview/"]
notoc: true
linktitle: Service Discovery
title: Service Discovery
weight: 10
menu:
  community_current:
    parent: "050 Backends Configuration"
---
Service discovery (`sd`) is an attribute in the `backend` section that enables KrakenD to detect and locate services automatically on your enterprise network.

The chosen service discovery strategy determines how to retrieve (statically or dynamically) the final list of IPs, hostnames, or services pointing to your backends. If your host list is dynamic, you can use an external service discovery provider and let KrakenD interact with it to get the hosts. If your host list is static (it doesn't change) or you use a service name or an external balancer, you can use `static` resolution and directly use the values provided under `host[]`.

## Static resolution
The `static` resolution is the default service discovery strategy. It implies that you write directly in the configuration the protocol plus the service name, hosts or IPs you want to connect to.

The `static` resolution uses a **list** of hosts to **load balance** (in a Round Robin fashion) all servers in the list, and you should expect more or less an equivalent number of hits on each backend. If you use a **Kubernetes service** that load balances itself.

KrakenD must be in a network where is able to reach all declared hosts. When there is more than one host, KrakenD load balances .

{{< note title="Declaring hosts on Kubernetes" type="tip" >}}
When the consumed hosts are behind a balancer or a service name, write a single entry in the array with that name.
{{< /note >}}

To use static resolution, you don't need to declare anything else than the `host` list, although you can also set the `"sd": "static"` property in the backend configuration (it is the default value when missing). Example:

{{< highlight json >}}
{
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
}
{{< /highlight >}}


## DNS SRV Service Discovery (Kubernetes/Consul)
The `DNS SRV` is a market standard used by systems such as **Kubernetes, Mesos, Haproxy, Nginx plus, AWS ECS, Linkerd**, and more.

To integrate Consul, Kubernetes or any other `DNS SRV` compatible system as the Service Discovery, you only need to set two keys:

- `"sd": "dns"`: To set service discovery = DNS SRV
- `"host": []`: The list of all the names providing the resolution

These keys need to be added in the `backend` section of your configuration. The `host` key can be skipped if there is another `host` key in the root level of the configuration.

For instance:

{{< highlight json >}}
{
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
}
{{< /highlight >}}

## Eureka (custom build)
Users of the Netflix's service [Eureka](https://github.com/Netflix/eureka) have a couple of user contributed integrations available listed in our [krakend-contrib](https://github.com/devopsfaith/krakend-contrib) repository.

These integrations are **not bundled with KrakenD** releases but they can be added to the project and make a custom build.

- [schibsted/krakend-eureka](https://github.com/schibsted/krakend-eureka): The Eureka client [Schibsted](https://www.schibsted.com) has been running in production since 2017.
- [joaoqalves/krakend-eureka](https://github.com/joaoqalves/krakend-eureka): A Eureka client contributed by [João Alves](https://twitter.com/joaoqalves)
