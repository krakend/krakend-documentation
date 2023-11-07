---
lastmod: 2021-12-02
date: 2017-01-21
linktitle: Running in a cluster
title: Clustering Deployment Guide
description: Discover the best practices for deploying KrakenD in a clustered environment. Scale your API infrastructure effectively using our comprehensive clustering deployment guide.
notoc: true
aliases: ["/docs/cluster/configuration/","/docs/cluster/cluster-overview/"]
menu:
  community_current:
    parent: "110 Deployment and Go-Live"

weight: 1
---
KrakenD nodes **are stateless** and they don't store data or application state to a persistent storage. Instead, any configuration data and application state exist within the configuration file. Nodes are expendable and replaceable at any time, as they do not hold anything.

A KrakenD cluster consists of multiple KrakenD instances running simultaneously and working together to provide increased reliability, higher throughput, scalability, and fail-over. There is no special software that you need to run a cluster other than KrakenD and the hardware or software that will balance the connections.

![Load balancing KrakenD](/images/documentation/diagrams/load-balancing-to-krakend.mmd.png)

## KrakenD cluster benefits
Having a KrakenD cluster provides these immediate **benefits**:

- **Increased throughput and capacity**: Having more KrakenD nodes expands the number of requests the API can handle.
- **Plug-and-play nodes**: Adding more nodes does not require any coordination or process, spin more servers on the go without interruption of the service.
- **Infinite scalability and zero coordination**: Unlike other gateways with shared state (and centralized coordination), every KrakenD node is stateless. Expect distributed, autonomous and independent nodes that do not communicate or coordinate between themselves, providing you infinite scalability.
- **High-Availability and Failover**: If for any reason a server dies or the instance fails, the rest of the members in the cluster continue providing the service without affecting the global availability.
- **Monitoring**: All KrakenD nodes individually report to an InfluxDB, Prometheus or any other of the available integrations you choose. Your metric collection systems will have the aggregated metrics of all the nodes.
- **No single point of failure**: A completely distributed cluster without any external dependencies that can shut down the gateway (e.g., a database failure)
- **Simple provision and maintenance**: A cluster that only requires spinning the servers with a copy of the same configuration file, it's impossible to have a simpler solution.

## Running the cluster
Running a cluster of machines is a straightforward process that only requires two conditions:

- Having a [balancer in front of the machines](/docs/throttling/load-balancing/) (e.g., ELB, Haproxy, Kubernetes...)
- Run two or more KrakenD services with the same configuration file

If you are in the cloud, something like an [ELB](https://aws.amazon.com/elasticloadbalancing) or equivalent does the job. On-premises users can use [HAProxy](http://www.haproxy.org/). When you have your load balancer in place, register all the KrakenD instances so they can start receiving traffic.

When all the desired nodes of KrakenD run, every instance honors its config and reports the traces and metrics to the services of your choice.

For best practices deploying KrakenD, see [Deploying KrakenD](/docs/deploying/)
