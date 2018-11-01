---
lastmod: 2018-11-01
date: 2017-01-21
linktitle: Cluster Overview
notoc: true
menu:
  documentation:
    parent: cluster
title: High-availability cluster
weight: 1
---
KrakenD is a stateless server and has been designed to avoid any kind of coordination between its nodes. A KrakenD cluster consists of multiple KrakenD instances running simultaneously and working together to provide increased reliability, higher throughput, scalability, and fail-over.

Having a KrakenD cluster provides these immediate **benefits**:

- **Increased throughput**: The KrakenD cluster does not limit the number of requests the service can deliver. And allows you
 to add as many nodes as you need
- **Scalability**: The capacity of an application deployed on a KrakenD cluster can be increased dynamically to meet demand as you can add KrakenD instances to the cluster without interruption of the service.
- **High-Availability and Failover**: If for any reason a server dies or the instance fails the rest of the members in the cluster continue providing the service without affecting the global availability.
- **Load Balancing**: To do an even distribution of requests to the different KrakenD instances.
- **Control**: All KrakenD nodes will report to InfluxDB, Prometheus or any other of the available integrations you choose.
