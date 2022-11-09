---
lastmod: 2020-03-27
date: 2017-01-21
linktitle: Best practices
aliases: ["/docs/deploying/best-practices/"]
menu:
  community_current:
    parent: "110 Deployment and Go-Live"
title: Production best practices
weight: 1
---
Setting up KrakenD is a straightforward process, but here are some not-so-obvious recommendations to get a good start when going live. This section has generalistic advice, despite that every KrakenD installation is different. Let's dip the toe into the deployment waters!

## Architecture recommendations
### High Availability
Hardware can fail at any time, and a Gateway is a piece critical enough to have redundancy of the service. Having a cluster of machines operating the service assures high availability. You should always plan to have at least a couple of KrakenD servers/containers running in case one of them gets in trouble, even when you have low traffic.

KrakenD can run in different regions and datacenters transparently without any problem as its nodes do not need to communicate to each other.

[Setup a cluster of machines](/docs/deploying/clustering/)

### Place a balancer in front of KrakenD
Put a [load balancer](/docs/throttling/load-balancing/) in front of KrakenD to distribute traffic between the different nodes of the cluster (Kubernetes already does this for you). Use always at least two KrakenD instances for High Availability.

### Server dimensioning
Dimension KrakenD nodes according to your expected needs and throughput.

See [server requirements](/docs/deploying/server-dimensioning/)

### Use several gateways
The API gateway doesn't need to be unique. We recommend using an independent KrakenD installation per consumer type. For instance, your iOS development team might need its own KrakenD with different views of the consumed content compared to the Web Team. Needs and content in each team differs in each endpoint, and every team could optimize the contract for each case.

### Use HTTP2
Whenever possible, enable HTTP2 between your balancer and KrakenD API gateway for the best performance. There is nothing additional you need to configure in KrakenD.

### SSL Certificates
Even that you can start KrakenD with SSL, you can add your public SSL certificate in the load balancer or PaaS and use **internal certificates**, or even no certificates at all (termination), between the [load balancer](/docs/throttling/load-balancing/) and KrakenD.

### Prepare for failure
Add a [circuit breaker](/docs/backends/circuit-breaker/) to your backends to avoid KrakenD keep pushing a failing system and throttle down for a while. If you know that a certain backend does not support more than a number of requests, add a maximum number of requests using the [proxy rate limit](/docs/backends/rate-limit/).

## Monitoring
### Enable traces and metrics
Make sure you have visibility of what is going on. Choose any of the systems where you can send the metrics and enable them. There are many choices, but choose wisely and do not enable them all!. If you don't use a SaaS provider, **a good self-hosted start** would be:

- A [Grafana Dashboard](/docs/telemetry/grafana/)
- Fueled by [InfluxDB](/docs/telemetry/influxdb/)
- And full traces by [Jaeger](/docs/telemetry/jaeger/)

Pay attention to the cardinality of the metrics. Logs and metrics might produce a lot of data and CPU activity. Aggregate and consolidate data in InfluxDB (e.g: When looking at the past year metrics, you don't need minute resolution and days will be enough).

### Add logging
If you don't add any logging, KrakenD will spit on stdout all the activity of the gateway. This behavior is not recommended for production. Enable the [logging](/docs/logging/) with `CRITICAL`, `ERROR` or `WARNING` levels at most. Avoid `INFO` and `DEBUG` levels in production at all times. This is the **recommended configuration** in production for a good performance:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "CRITICAL",
      "syslog": false,
      "stdout": false
    }
  }
}
```


Send logs to an [ELK](/docs/logging/logstash/), the [syslog](/docs/logging/#write-to-syslog-or-stdout), or a [GELF server](/docs/logging/graylog-gelf/).

{{< note title="Redirect ouput to /dev/null for maximum performance" type="tip" >}}
**When the output of KrakenD stdout is not important to you**, set the logging level to `CRITICAL` and redirect its output to `/dev/null` to have even more performance. To do that, start KrakenD with:

    krakend run -c krakend.json >/dev/null 2>&1
{{< /note >}}


## Deployment recommendations

### Release through a CI/CD pipeline
Automate the go-live process through a [CI/CD pipeline](/docs/deploying/ci-cd/) that builds and checks KrakenD configuration before deploying.

### Use Docker and immutable containers
On Docker deployments, creating an immutable Docker image with your desired configuration takes a few seconds in your CI/CD pipeline. Create a `Dockerfile` with at least the following code and deploy the resulting image in production:

```Dockerfile
FROM {{< product image >}}:{{< product latest_version >}}
COPY krakend.json /etc/krakend/krakend.json
```

Read more on [Docker artifacts](/docs/deploying/docker/)

### Use blue/green or similar deployment strategy
As it happens with Apache, Nginx, Varnish and other stateless services, changing the configuration requires a restart. When deploying new changes, use a technique like blue/green deployment or similar to make the deploy transparent for the user.

This scenario can be automated and is available in Kubernetes and in all major cloud providers. The idea is that you spin up new machines with the latest configuration and then shift the traffic from the old instances to the new ones.

This methodology ensures that there is no downtime when applying changes. On-premises installations can make a similar approach as well, but the implementations depends on the underlying infrastructure.

## Code organization
### Name your configurations
Add a `name` key in the configuration file with useful information so you can identify which specific version your cluster is running. Whatever type of information you write inside the `name` is open to your imagination. Any value you write is **available in the metrics** for inspection.

```json
{
    "version": 3,
    "name": "Production Cluster rev-db6a182"
}
```


**During the build in the pipeline**, it might be a good idea to **replace the content** of the `name` attribute by a content showing the deployed version (the short SHA from the commit maybe).

### Add comments and metadata  (`@`)
During startup, KrakenD **ignores from the configuration anything that it doesn't recognize**. Meaning that your `krakend.json` (or whatever format you use) allows you to include additional metadata and fields that make sense to your company. Use it to add your meta language, tags, comments, bot integrations, etc. for better integration with your CI/CD system, deployment process, or just better comprehension of the file in the future.

{{< note title="Validating KrakenD's schema" type="tip" >}}
If you use the KrakenD `$schema` to validate your configuration, unknown attributes will trigger a warning during validation. To add your own configurations schema-compatible, and have them ignored by KrakenD, prefix them with one of the following characters: `@`, `$`, `_` or `#`.
{{< /note >}}

For instance, you could add `@comment` fields. The field is not used by KrakenD and it passes the JSON schema validation. Finding it might be fresh air for the developer next to you.


{{< highlight json "hl_lines=4" >}}
{
    "endpoint": "/cookies",
    "input_headers": ["Cookie" ],
    "@comment": "At this early stage of the implementation, we still need to send cookies to the backend.",
    "backend": [{
        "url_pattern": "/srv/legacy"
    }]
}
{{< /highlight >}}

### Split the configuration in multiple repos or folders
On large organizations with several teams using a common gateway, you might want to split the endpoints in groups using folders or even different repositories. With the [flexible configuration](/docs/configuration/flexible-config/) you can have teams working in its dedicated space and aggregate all endpoints during build time without conflicts touching the same files.

Most KrakenD configurations tend to be large and with repetitive blocks. Define a basic skeleton of configurations that will be used across all teams.
