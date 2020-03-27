---
lastmod: 2018-11-02
date: 2017-01-21
linktitle: Setting up a cluster
menu:
  documentation:
    parent: cluster
title: Setting up a cluster
weight: 10
notoc: true
---
Hardware can fail at any time, and a Gateway is a piece critical enough to have redundancy of the service. Having a cluster of machines operating the service assures high availability.

KrakenD nodes **are stateless** and they don't store data or application state to a persistent storage. Instead, any configuration data and application state exist within the configuration file. Nodes are expendable and replaceable at any time, as they do not hold anything.

# Running the cluster
Running a cluster of machines is a straightforward process that only requires two conditions:

- Having a balancer in front of the machines (e.g., ELB, Haproxy, Kubernetes...)
- Run two or more KrakenD services with the same configuration file

If you are in the cloud, something like an [ELB](https://aws.amazon.com/elasticloadbalancing) or equivalent does the job. On-premises users can use [HAProxy](http://www.haproxy.org/). When you have your load balancer in place, register all the KrakenD instances so they can start receiving traffic.

When all the desired nodes of KrakenD run, every instance honors its config and reports the traces and metrics to the services of your choice.

For best practices deploying KrakenD, see [Deploying KrakenD](/docs/deploying/best-practices)