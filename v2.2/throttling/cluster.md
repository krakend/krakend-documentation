---
lastmod: 2022-01-28
old_version: true
date: 2022-01-28
linktitle: Cluster rate-limit
title: Stateless rate-limiting on clusters
weight: 100
notoc: true
menu:
  community_v2.2:
    parent: "070 Traffic Management"
---
The stateless rate-limiting is the recommended approach for almost all scenarios. As the API Gateway does not have any centralization, **the limits apply individually to each running instance of KrakenD**.

{{< note title="Global rate limit" type="info" >}}
If you prefer not to use a stateless rate limit, the KrakenD Enterprise edition has a [stateful Redis-backed rate limit](/docs/v2.2/enterprise/throttling/global-rate-limit/) where counters are shared amongst all nodes.
{{< /note >}}

Working in a cluster implies applying the limits taking into account the deployment size. For instance, if you want to use a limit of 100 reqs/s in a specific endpoint, the configuration will need to consider the number of nodes in the cluster.

Let's say that you have a cluster deployed with three instances of KrakenD. If you write a `max_rate=100` in the configuration, your ecosystem is limiting to 300 reqs/s in total as three servers are receiving balanced traffic.

When you have to do small math like this, through the [Flexible Configuration](/docs/v2.2/configuration/flexible-config/), you might inject environment variables when starting KrakenD with the total number of machines you have. For instance:

```tpl
{
    "endpoint": "/limited-endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "max_rate": {{ env "NUM_PODS" | div 100 }}
        }
    }
}
```

You can execute krakend like this:

{{< terminal title="Term" >}}
FC_ENABLE=1 \
FC_OUT=compiled.json \
NUM_PODS=3 krakend check -c krakend.json
{{< /terminal >}}


And you get in the compiled.json the following content:

```json
{
    "endpoint": "/limited-endpoint",
    "extra_config": {
      "qos/ratelimit/router": {
          "max_rate": 33
        }
    }
}
```
