---
lastmod: 2022-11-11
old_version: true
date: 2017-01-21
linktitle: Best practices
menu:
  community_v2.3:
    parent: "110 Deployment and Go-Live"
title: Production best practices
weight: 1
---
Setting up KrakenD is a straightforward process, but here are some not-so-obvious recommendations to get a good start when going live. This section has generalistic advice, even though every KrakenD installation is different. So let's dip our toe into the deployment waters!

## Architecture recommendations
### High Availability
Hardware can fail anytime, and a Gateway is critical enough to have redundancy. Having a cluster of machines operating the service assures high availability. It would be best if you always planned to have at least a couple of KrakenD servers/containers running in case one of them gets in trouble, even when you have low traffic.

KrakenD can run in different regions and data centers transparently without any problem, as its nodes do not need to communicate with each other.

[Setup a cluster of machines](/docs/v2.3/deploying/clustering/)

### Place a balancer in front of KrakenD
Put a [load balancer](/docs/v2.3/throttling/load-balancing/) in front of KrakenD to distribute traffic between the different nodes of the cluster (Kubernetes already does this for you). Use always at least two KrakenD instances for High Availability.

### Server dimensioning
Dimension KrakenD nodes according to your expected needs and throughput.

See [server requirements](/docs/v2.3/deploying/server-dimensioning/)

### Use several gateways
The API gateway doesn't need to be unique. We recommend using an independent KrakenD installation per consumer type. For instance, your iOS development team might need a gateway with different views of the consumed content compared to the Web Team. Needs and content in each team differ in each endpoint, and every team could optimize the contract for each case.

### Use HTTP2
Enable HTTP2 between your balancer and KrakenD API gateway for the best performance. There is nothing additional you need to configure in KrakenD.

### SSL Certificates
Even that you can start KrakenD with SSL, you can add your public SSL certificate in the load balancer or PaaS and use **internal certificates**, or even no certificates at all (termination), between the [load balancer](/docs/v2.3/throttling/load-balancing/) and KrakenD.

### Prepare for failure
Add a [circuit breaker](/docs/v2.3/backends/circuit-breaker/) to your backends to prevent KrakenD from pushing to a failing system. In addition, if you know that a particular backend does not support more than a number of requests, add a maximum number of requests using the [proxy rate limit](/docs/v2.3/backends/rate-limit/).

## Configuration recommendations
KrakenD will behave according to the configuration file, here are some recommendations:

### Add logging and suppress unnecessary information
If you don't add any logging, KrakenD will spit on stdout all the activity of the gateway. This behavior is not recommended for production.

Enable the [logging](/docs/v2.3/logging/) with `CRITICAL`, `ERROR`, or `WARNING` levels at most. Avoid `INFO` and `DEBUG` levels in production at all times.

Below there is a **recommended configuration** in production for a good performance:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "ERROR",
      "syslog": false,
      "stdout": true
    }
  }
}
```
If you send the logs out to an [ELK](/docs/v2.3/logging/logstash/) or a [GELF server](/docs/v2.3/logging/graylog-gelf/), use `"stdout": false` to avoid having double output.

{{< note title="Redirect ouput to /dev/null for maximum performance" type="tip" >}}
**When the output of KrakenD stdout is not important to you**, set the logging level to `CRITICAL` and redirect its output to `/dev/null` to have even more performance. To do that, start KrakenD with:

    krakend run -c krakend.json >/dev/null 2>&1
{{< /note >}}


### Remove access logs
Removing the access log increases the number of requests per second the gateway can serve on high concurrency.**[Disable the access log](/docs/v2.3/service-settings/router-options/#disable_access_log)** to gain more speed. You will still have the problems logged during runtime, but the requests won't be outputted.
```json
{
  "version": 3,
  "extra_config": {
    "router": {
       "disable_access_log": true
    }
  }
}
```

### Cache JWT keys
If you use token validation and use a `jwk_url` (instead of a ` jwk_local_path`), enable the caching. Otherwise, each endpoint will require to download the JWK over HTTP for every request, e.g.:
```json
{
    "endpoint": "/protected/resource",
    "extra_config": {
        "auth/validator": {
            "alg": "RS256",
            "jwk_url": "https://some/.well-known/jwks.json",
            "cache": true
        }
    }
}
```

## Monitoring
### Enable traces and metrics
Make sure you have visibility of what is going on. Choose any of the systems where you can send the metrics and enable them. There are many choices, but choose wisely and do not enable them all!. If you don't use a SaaS provider, **a good self-hosted start** would be:

- A [Grafana Dashboard](/docs/v2.3/telemetry/grafana/)
- Fueled by [InfluxDB](/docs/v2.3/telemetry/influxdb/)
- And full traces by [Jaeger](/docs/v2.3/telemetry/jaeger/)

Pay attention to the cardinality of the metrics. Logs and metrics might produce a lot of data and CPU activity. Aggregate and consolidate data in InfluxDB (e.g., when looking at the past year's metrics, you don't need minute resolution, and days will be enough).

## Deployment recommendations

### Release through a CI/CD pipeline
Automate the go-live process through a [CI/CD pipeline](/docs/v2.3/deploying/ci-cd/) that builds and checks KrakenD configuration before deploying.

At least your pipeline should have:

- `krakend check -d -t -c krakend.[tmpl|json|yml]`
- `krakend check -c krakend.json --lint`

If you don't use flexible configuration, you can do it all in one line: `krakend check -d - t -c krakend.json --lint`

It is also important adding the [audit command](/docs/v2.3/configuration/audit/), excluding any rules that do not apply to you:

- `krakend audit -c krakend.json`

### Use Docker and immutable containers
Creating an immutable Docker image with your desired configuration takes a few seconds in your CI/CD pipeline on Docker deployments. Create a `Dockerfile` with at least the following code and deploy the resulting image in production:

```Dockerfile
FROM {{< product image >}}:2.3
COPY krakend.json /etc/krakend/krakend.json
```

Read more on [Docker artifacts](/docs/v2.3/deploying/docker/)

### Use blue/green or similar deployment strategy
As with Apache, Nginx, Varnish, and other stateless services, changing the configuration require a restart. When deploying new changes, use a technique like blue/green deployment or similar to make the deployment transparent for the user.

This scenario can be automated and is available in Kubernetes and in all major cloud providers. The idea is that you spin up new machines with the latest configuration and then shift the traffic from the old instances to the new ones.

This methodology ensures that **there is no downtime when applying changes**. Of course, on-premises installations can also make a similar approach, but the implementations depend on the underlying infrastructure.

## Code organization
### Name your configurations
Add a `name` key in the configuration file with helpful information to identify your cluster's specific version. Whatever type of information you write inside the `name` is open to your imagination. Any value you write is **available in the metrics** for inspection.

```json
{
    "version": 3,
    "name": "Production Cluster rev-db6a182"
}
```


**During the build in the pipeline**, it might be a good idea to **replace the content** of the `name` attribute with content showing the deployed version (the short SHA from the commit, maybe).

### Add comments and metadata  (`@`)
During startup, KrakenD **ignores from the configuration anything that it doesn't recognize**, which means that your `krakend.json` (or whatever format you use) allows you to include additional metadata and fields that make sense to your company. Use it to add your meta language, tags, comments, bot integrations, etc., for better integration with your CI/CD system, deployment process, or a better future comprehension of the file.

{{< note title="Validating KrakenD's schema" type="tip" >}}
If you use the KrakenD `$schema` to validate your configuration, unknown attributes will trigger a warning during validation. To add your configurations schema-compatible, and have them ignored by KrakenD, prefix them with one of the following characters: `@`, `$`, `_` or `#`.
{{< /note >}}

For instance, you could add `@comment` fields. KrakenD does not use the field, and it passes the JSON schema validation. Finding it might be fresh air for the developer next to you.


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
In large organizations with several teams using a shared gateway, you can split the endpoints into groups using folders or even different repositories. With the [flexible configuration](/docs/v2.3/configuration/flexible-config/), you can have teams working in its dedicated space and aggregate all endpoints during build time without conflicts touching the same files.

Most KrakenD configurations tend to be large and with repetitive blocks. Define a basic skeleton of configurations that will be used across all teams.
