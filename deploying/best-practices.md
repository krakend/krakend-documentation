---
lastmod: 2018-11-02
date: 2017-01-21
linktitle: Best practices
menu:
  documentation:
    parent: deploying
title: Deployment best practices
weight: 10
---

<span class="badge badge-warning">This document is a draft</span>

Setting up a cluster of KrakenD instances is a straightforward process but here are some recommendations to get a good start.

# Use HTTP2
Enable HTTP2 between your balancer and KrakenD API gateway for the best performance.

# SSL
Add your SSL certificate in the load balancer and use **internal certificates** between the load balancer and KrakenD.

# Enable metrics and logging
Make sure you have visibility of what is going on. Choose any of the systems where you can send the metrics and enable them. Enable logging with at least a `WARNING` level.

# Startup command
Redirect the output to `/dev/null`.  Run the service with

	krakend run -c krakend.json >/dev/null 2>&1

# Named configuration
Add a `name` key in the configuration file with useful information so you can identify which specific version your cluster is running. E.g.:

    {
        "version": 2,
        "name": "Production Cluster rev-db6a182"
    }

Whatever type of information you write inside the `name` is open to your imagination. Any value you write is available in the metrics for inspection.
