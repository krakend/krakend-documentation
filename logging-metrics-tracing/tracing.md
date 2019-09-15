---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Tracing
title: Tracing
weight: 30
menu:
  documentation:
    parent: logging-metrics-tracing
---
Moving from a single monolithic application to a distributed microservices architecture presents a new type of challenges. Observability and networking are key to succeed in this new scenario, and new monitoring tools are needed. These tools must provide at least options to detect problems' root causes, monitoring and details of the different distributed transactions, and performance and latency optimization.

The [Opencensus exporters](/docs/logging-metrics-tracing/opencensus/) allow you to send traces to several of these open source and privative tools, so you can follow the activity of the Gateway and the derived requests to the backends.

Enabling tracing is key to have a good detail of what is going on inside the Gateway and between the user, the Gateway and your services.

The following tracing exporters are supported:

- Jaeger
- Zipkin
- AWS X-Ray