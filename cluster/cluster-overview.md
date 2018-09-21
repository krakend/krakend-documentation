---
lastmod: 2017-01-21
date: 2017-01-21
linktitle: Overview
notoc: true
menu:
  documentation:
    parent: cluster
title: Cluster Management Overview
weight: 1
---

<div class="alert alert-warning alert-dismissible">
    <button type="button" class="close" data-dismiss="alert" aria-hidden="true">×</button>
    <p><i class="icon fa fa-warning"></i> Alert!</p>
    The cluster version is not available in the open source distribution.
</div>

A KrakenD cluster consists of multiple KrakenD instances running simultaneously and working
together to provide increased reliability, higher throughput, scalability, and fail-over.

The KrakenD cluster provides these immediate **benefits**:

- **Increased throughput**: The KrakenD cluster does not limit the number of requests the service can deliver. And allows
 to add several nodes as you need (see your subscription)
- **Scalability**: The capacity of an application deployed on a KrakenD cluster can be increased dynamically to meet demand as you can add
KrakenD instances to the cluster without interruption of the service.
- **High-Availability and Failover**: If for any reason a server dies or the instance fails the rest of the members in the cluster continue providing the service
without affecting the global availability.
- **Load Balancing**: To do an even distribution of requests to the different KrakenD instances.
- **Control**: As the Cluster Manager comes with a web interface to see the activity of the cluster.

# Introducing the Cluster Manager
The Cluster Manager administers all the nodes that provide the same KrakenD service in the cluster. It manages cluster
membership and member failure detection using a gossip based protocol (UDP).  Node failures are detected
and network partitions are partially tolerated by attempting to communicate to potentially dead nodes through multiple routes.

A KrakenD cluster uses the **KrakenD Cluster Manager** to coordinate the formation of the cluster. The Cluster Manager
is an independent binary with the following responsibilities:

- Validates your subscription level and validates the license
- Keeps all the KrakenDs configuration up to date
- Restarts gracefully the services when needed
- Collects and centralizes all the stats sent by the different KrakenD nodes
- Administers the membership of the nodes and acts as seeder (leader) of the cluster

Once all the nodes are registered in the cluster, the only communication with the Cluster Manager is to send the stats and
metrics and to collect configuration updates.

If for any reason the cluster manager is stopped the KrakenD nodes will continue to run anyway in cluster as they recognize themselves
as a cluster. But if after several days the nodes cannot connect with the manager all the nodes will degrade to the *free version*
with all the corresponding limits and will un-register themselves from the cluster.

# KrakenD Nodes
The binary for the KrakenD nodes it's the same as the standalone. So any standalone installation could be at some point used to be included
inside a cluster with a simple change in its configuration file.


# Web interface: Cluster Manager
The cluster manager machine exposes by default a web interface in the port `9897`. The activity of your cluster can be
followed in a URL like **[http://localhost:9897](http://localhost:9897)** (use your own IP)

The web interface shows the activity to all your nodes and to your backends.
