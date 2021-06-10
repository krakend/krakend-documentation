---
lastmod: 2018-11-02
date: 2017-01-21
linktitle: Cluster Overview
notoc: true
menu:
  community_v1.3:
    parent: "120 Cluster"
title: High-availability cluster
weight: 1
---
A KrakenD cluster consists of multiple KrakenD instances running simultaneously and working together to provide increased reliability, higher throughput, scalability, and fail-over.

A KrakenD cluster runs with the same KrakenD open source software you use today to start a single instance. Consequently **no license is needed** to operate a sizeable enterprise-grade API gateway.

## KrakenD cluster benefits
Having a KrakenD cluster provides these immediate **benefits**:

- **Increased throughput and capacity**: Having more KrakenD nodes expands the number of requests the API can handle.
- **Plug-and-play nodes**: Adding more nodes does not require any coordination or process, spin more servers on the go without interruption of the service.
- **Infinite scalability and zero coordination**: Unlike other gateways with shared state (and centralized coordination), every KrakenD node is stateless. Expect distributed, autonomous and independent nodes that do not communicate or coordinate between themselves, providing you infinite scalability.
- **High-Availability and Failover**: If for any reason a server dies or the instance fails, the rest of the members in the cluster continue providing the service without affecting the global availability.
- **Monitoring**: All KrakenD nodes individually report to an InfluxDB, Prometheus or any other of the available integrations you choose. Your metric collection systems will have the aggregated metrics of all the nodes.
- **No single point of failure**: A completely distributed cluster without any external dependencies that can shut down the gateway (e.g., a database failure)
- **Simple provision and maintenance**: A cluster that only requires spinning the servers with a copy of the same configuration file, it's impossible to have a simpler solution.
