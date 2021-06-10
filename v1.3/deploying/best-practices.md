---
lastmod: 2020-03-27
date: 2017-01-21
linktitle: Best practices
menu:
  community_v1.3:
    parent: "110 Deploying"
title: Deployment best practices
weight: 10
---

---
Setting up a cluster of KrakenD instances is a straightforward process, but here are some not so obvious recommendations to get a good start.


## Use blue/green or similar deployment strategy
As it happens with Apache, Nginx, Mysql, and the vast majority of services, changing the configuration requires a restart. When deploying new changes, use a technique like blue/green deployment or similar.

This scenario can be automated and is available in all major cloud providers. The idea is that you spin up new machines with the latest configuration and then shift the traffic from the old instances to the new ones.

This methodology ensures that there is no downtime when applying changes. On-premises installations can make a similar approach as well.

## Use Docker and immutable containers
Creating an immutable Docker image with your desired configuration takes a few seconds in your CI/CD pipeline. Create a `Dockerfile` with at least the following code and deploy the resulting image in production:

    FROM devopsfaith/krakend

    COPY krakend.json /etc/krakend/krakend.json

## Place a balancer in front of KrakenD
Put a load balancer in front of KrakenD to distribute traffic between the different nodes of the cluster (Kubernetes already does this for you). Use always at least two KrakenD instances for High Availability.

## Use HTTP2
Enable HTTP2 between your balancer and KrakenD API gateway for the best performance.

## SSL
Add your public SSL certificate in the load balancer and use **internal certificates** (or even no certificates at all -termination-) between the load balancer and KrakenD.

## Enable metrics and logging
Make sure you have visibility of what is going on. Choose any of the systems where you can send the metrics and enable them. Enable logging with at least a `WARNING` level.

## Startup command
Redirecting the output to `/dev/null` gives you more performance.  Run the service with:

    krakend run -c krakend.json >/dev/null 2>&1

## Named configuration
Add a `name` key in the configuration file with useful information so you can identify which specific version your cluster is running. Whatever type of information you write inside the `name` is open to your imagination. Any value you write is available in the metrics for inspection.

E.g.:

    {
        "version": 2,
        "name": "Production Cluster rev-db6a182"
    }

**During the build in the pipeline**, it might be a good idea to **replace the content** of the `name` attribute by a content showing the deployed version (the short SHA from the commit maybe).

## Add metadata and your company attributes
KrakenD **ignores anything that it doesn't recognize** in the configuration as its own. Meaning that your `krakend.json` (or whatever format you use) is open to include additional metadata and fields that make sense to your company. Use it to add your meta language, tags, comments, bot integrations, etc. for better integration with your CI/CD system, deployment process, or just better comprehension of the file in the future.

If you add new attributes to the configuration file, make sure they don't collide with new features in the future.

For instance, we are adding a `@comment` field in the following configuration. This attribute is ignored by KrakenD:


{{< highlight json "hl_lines=4" >}}
{
    "endpoint": "/cookies",
    "headers_to_pass": ["Cookie" ],
    "@comment": "At this early stage of the implementation, we still need to send cookies to the backend.",
    "backend": [{
        "url_pattern": "/srv/legacy"
    }]
}
{{< /highlight >}}
