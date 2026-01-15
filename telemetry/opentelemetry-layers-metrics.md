---
lastmod: 2024-03-01
date: 2024-01-22
notoc: false
linktitle: OpenTelemetry - Layers and Metrics
title: Understanding OpenTelemetry layers and metrics
description: Learn about the metrics available when activating OpenTelemetry on KrakenD API Gateway, enabling real-time visibility and analysis of API performance
weight: 21
#images:
#- /images/documentation/available-exporters.png
menu:
  community_current:
    parent: "160 Monitoring, Logs, and Analytics"
---
You can add several `exporters` to your [OpenTelemetry configuration](/docs/telemetry/opentelemetry/#opentelemetry-configuration) (the more, the hungrier the gateway will be), and KrakenD will send data to all the declared exporters and layers by default.

While `exporters` define **where** you will have the metrics, the `layers` define **which** metrics you want to have. The layers contain the traces and metrics for a subset of the execution flow. These are the layers you can use:

![Diagram showing global, proxy, and backend sequence](/images/documentation/diagrams/opentelemetry-layers.mmd.svg)

- `global`: The global layer contains everything that KrakenD saw in and out. It includes the total timings for a request hitting the service until it is delivered to the client.
- `proxy`: The proxy layer starts acting after processing the HTTP request and comprehends the internal work of dealing with the endpoint, including spawning multiple requests to the backends, aggregating their results, and manipulating the final response.
- `backend`: The backend layer monitors the activity of a single backend request and response with any additional components in its level. It contains mainly the timings between KrakenD and your service. It is the richest layer of all.

**It is vital to see that `backend` is a subset of `proxy`, and `proxy` is a subset of `global`**. Do not generate metrics for the layers you will not use.

{{< note title="CPU and memory consumption" type="warning" >}}
As you will see below, the amount of telemetry data you can send for each request is massive. The more layers and details you add, the more resources KrakenD will consume and the more storage you will need to save metrics and traces. Choose carefully a balance between observability power and resource consumption to save money!

In production, **you should only enable a small sample rate of the traces** (e.g., 5%). The data generated in telemetry is usually more extensive than the responses of the services you expose.
{{< /note >}}

Once you configure the OpenTelemetry component, KrakenD will start reporting metrics when there is activity. The section below describes different metrics in each layer, briefly explaining their meaning.

**The presence of the metrics and the traces described below depend on the [OpenTelemetry configuration](/docs/telemetry/opentelemetry/#opentelemetry-configuration).**

{{% otel_metrics %}}


## Measuring added latency and similar metrics
A common metric developers like to measure is the gateway's added latency and other similar metrics. In order to get coherent results you must take into account that:

* Having more than one backend in a single endpoint will produce more than one record for the `http.client.duration` metric.
* Requests handled by KrakenD might not reach backends.

So, to find the time spent in KrakenD, subtract the **sum** of `http.client.duration` from the **sum** of `http.server.duration` **only for successful responses** (200 OK) on **endpoints with a single backend**, and divide by the number of received requests. This yields an average per-request added latency attributable to KrakenD.

This is a restrictive scenario, so unless you want to continuously monitor that metric, you should enable traces (with sampling) and inspect a few requests to get a better idea of added latency per endpoint. Traces provide a tree view showing how much time is spent in each “stage” of the request and give more actionable insight than aggregated metrics alone.

Let's deep dive a little...

Incoming requests are handled at the global layer first. There are many factors that determine whether they reach the backend or not. The gateway is meant to prematurely terminate illegitimate traffic, so **you will never have the same number of records in the global layer as in the other** layers. For example, a not-found 404 route, an invalid JWT token, a violated policy, or a server plugin raising an error, to name a few examples, are examples of records that you can see only in the global layer. As they do not succeed, the request does not produce a record for the proxy or backend layers and your upstream services were never reached in those cases.

In other words, **there is never a 1:1 relationship between the global and proxy layers**, because some requests will not pass through the global layer. Those requests are handled very quickly (under a millisecond), which will skew averages if mixed with normal proxy/backend timings.

Another aspect to have into account is that KrakenD is not a simple proxy, so an endpoint might have more than one backend request.

The interpretation of metrics can also be misleading when an endpoint spawns multiple parallel backend requests. For example, if you have two backend requests and one takes `10 ms` and the other `100 ms`, the naive average of those two backend durations is `55 ms`, but the proxy layer will report a single duration that is higher than `100 ms` (because it waits for all backends and aggregates). That could lead you to incorrectly assume the proxy layer is adding more latency than it actually is.

Traces and sampling are the best way to validate and understand these cases: a few sampled traces will show the timings for each backend call and the proxy's aggregation time.