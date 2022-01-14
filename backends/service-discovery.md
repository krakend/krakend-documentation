---
lastmod: 2022-01-14
date: 2016-09-30
aliases: ["/service-discovery/dns-srv/", "/docs/service-discovery/overview/"]
linktitle: Service Discovery
title: Service Discovery
weight: 10
menu:
  community_current:
    parent: "050 Backends Configuration"
---
Service discovery (`sd`) is an attribute in the `backend` section that enables KrakenD to detect and locate services automatically on your enterprise network.

The chosen service discovery strategy determines how to retrieve (statically or dynamically) the final list of IPs, hostnames, or services pointing to your backends. If your host list is dynamic, you can use an external service discovery provider and let KrakenD interact with it to get the hosts. If your host list is static (it doesn't change) or you use a service name or an external balancer, you can use `static` resolution and directly use the values provided under `host[]`.

KrakenD must be in a network where can reach any declared hosts. When there is more than one host, KrakenD load balances the connections to the hosts in the list.

## Static resolution
The `static` resolution is the default service discovery strategy. It implies that you write directly in the configuration the protocol plus the service name, hosts or IPs you want to connect to.

The `static` resolution uses a **list** of hosts to **load balance** (in a Round Robin fashion) all servers in the list, and you should expect more or less an equivalent number of hits on each backend. However, if you use a **Kubernetes service**, then it load-balances itself so that you will need one entry only.

{{< note title="Declaring hosts on Kubernetes" type="tip" >}}
When the consumed hosts are behind a balancer or a service name, write a single entry in the array with that name.
{{< /note >}}

You don't need to declare anything other than the `host` list to use static resolution. However, you can also add the `"sd": "static"` property in the backend configuration. It is the default value when `sd` is not declared). Example:

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
The `DNS SRV`([RFC](https://datatracker.ietf.org/doc/html/rfc2782)) is a market standard used by systems such as **Kubernetes, Mesos, Haproxy, Nginx plus, AWS ECS, Linkerd**, and more. An SRV entry is a custom DNS record used to establish connections between services. When KrakenD needs to know the location of a specific service, it will search for a related SRV record.

The format of the `SRV` record is as follows:

    _service._proto.name. ttl IN SRV priority weight port target

**Example**. A service running on port `8000` with maximum priority (`0`) and a weight `5` ):

    _api._tcp.example.com. 86400 IN SRV 0 5 8000 foo.example.com.


To integrate Consul, Kubernetes or any other `DNS SRV` compatible system as the Service Discovery, you only need to set two keys:

- `"sd": "dns"`: To set service discovery = DNS SRV
- `"host": []`: The list of all the names providing the resolution

Add these keys in the `backend` section of your configuration. If there is another `host` key in the root level of the configuration, you don't need to declare it here if the value is the same.

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

### Priority and weight importance on balancing
The `SRV` record provides the hostname, port, priority, and weight that KrakenD uses to balance. KrakenD reads these values **every 30 seconds**and generates an internal balancing list.

The balancing list honors the distribution described in the `SRV` records. Nevertheless, KrakenD will use only the records with the **lower priority**. So, for instance, if you have 5 servers with priority `0` and another with priority `2`, the latter won't be included in the balancing.

As per the weights, KrakenD distributes the traffic in the proportion they represent. To be memory and space-efficient, KrakenD compacts and normalizes the final list of weights if needed. It's essential to be aware that KrakenD will remove servers with a weight **orders of magnitude inferior** (under 1% of the total representation) from the final list as they are negligible.

Some examples on the space optimization and removal of neglectable items:
- `SRV` passes the weights of 3 servers with values `[100 500 1000]` and KrakenD builds a list `[1 5 10]`
- `SRV` passes `[25 10000 1000]` and KrakenD compacts it as `[0 10 1]`. The server with a weight of `25` is removed as it is vastly inferior to the rest (0,2% weight).
- `SRV` passes `[25 1000 10000 65535]` becomes `[0 1 13 85]`. Again, `25` drops. The rest are converted to a lower value with the same distribution proportion.


## Eureka (custom build)
Users of the Netflix's service [Eureka](https://github.com/Netflix/eureka) have a couple of user contributed integrations available listed in our [krakend-contrib](https://github.com/devopsfaith/krakend-contrib) repository.

These integrations are **not bundled with KrakenD** releases, but they can be added to the project and make a custom build.

- [schibsted/krakend-eureka](https://github.com/schibsted/krakend-eureka): The Eureka client [Schibsted](https://www.schibsted.com) has been running in production since 2017.
- [joaoqalves/krakend-eureka](https://github.com/joaoqalves/krakend-eureka): A Eureka client contributed by [João Alves](https://twitter.com/joaoqalves)
