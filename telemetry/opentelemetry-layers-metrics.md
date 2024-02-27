---
lastmod: 2024-01-29
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

**It is vital to see that `backend` is a subset of `proxy`, and `proxy` is a subset of `global.`**. Do not generate metrics for the layers you will not use.

{{< note title="CPU and memory consumption" type="warning" >}}
The more layers and details you add, the more resources KrakenD will consume and the more storage you need to save the metrics and traces. Choose carefully a balance between observability power and resource consumption to save money!
{{< /note >}}


## Exposed metrics and traces
Once the OpenTelemetry component is properly configured, KrakenD will start reporting metrics when there is activity. This section describes the different metrics you have in each layer with a short explanation of their meaning.

### Metrics of the `global` layer

- `global-response-latency`: histogram of the time it takes to produce the response. Attributes:
    - `http.response.status_code`: status code of the produced response
    - `url.path`: the matched endpoint path
    - `krakend.stage`: always with value `global`
- `global-response-size`: histogram of the size of the body produced for the response. Attributes:
    - `http.response.status_code`: status code of the produced response
    - `url.path`: the matched endpoint path
    - `krakend.stage`: always with value `global`


### Metrics of the `proxy` layer

- `stage-duration`: histogram of the time it takes to produce the response.
    Attributes:
    - `url.path`: the matched endpoint path that **krakend is serving** (is different
        than in `backend`, krakend stage, when this property is the path
        for the backend we are targetting).
    - `krakend.stage`: always with value `proxy`

### Traces of the `proxy` layer
Attributes:

- `krakend.stage`: always with value `proxy`
- `complete`: a `true` / `false` value to know when a response is complete (all backends returned a successful response).
- `canceled`: if it appears, it will always be `true` and indicates a request
    that has been canceled (usually when parallel requests are used).
- `error`: in case an error happens, the error description.

### Metrics of the `backend` layer

- `stage-duration`: histogram of the time it takes to produce the response controlled by the `disable_stage` flag (if set to `true`, this metric will not appear).
    Attributes:
    - `url.path`: the matched endpoint path that **krakend is serving** (is different
        than in `backend`, krakend stage, when this property is the path
        for the backend we are targetting).
    - `krakend.stage`: always with value `backend`
    - `krakend.endpoint`: this attribute is set to the KrakenD exposed endpoint that is the "parent" of the backend request.
    - `server.address`: the target host (in case more than one is provided, those are joined with `_`).

#### Round Trip metrics
In addition, if you enable the `round_trip` in the `backend` layer, you also get the following:

- `http.request.method_original`: the method used for the request to the backend (GET, POST,...)
- `url.path`: the requested path
- `server.address`: the target host (the first in the list of provided hosts).
- `krakend.stage`: always with value `backend-request`

- `requests-started-count`: number of requests started.
- `requests-failed-count`: number of requests failed.
- `requests-canceled-count`: number of canceled requests.
- `requests-timedout-count`: number of timed-out requests.
- `requests-content-length`: counter with the sum of `Content-Length` header for the
    sent payload for the request.

- `response-latency`: histogram with the time taken until receiving the first byte of the response
- `response-content-length`: histogram with the size of response bodies as reported in the
    `Content-Lenght` header.


#### Read Payload metrics
The `read_payload` on the `backend` layer enables the following metrics:

- `read-size`: counted with the read bytes
- `read-size-hist`: histogram with the read bytes
- `read-time`: counter with seconds spent reading the body payload of the response.
- `read-time-hist`: histogram with the seconds spent reading the body.
- `read-errors`: a counter of the number of errors that happened reading the response body.

#### Detailed connection metrics
The `detailed_connection` flag on the `backend` layer enables the following metrics:

- `http.request.method_original`: the method used for the request to the backend
    (GET, POST,...)
- `url.path`: the requested path
- `server.address`: the target host (the first in the list of provided hosts).
- `krakend.stage`: always with value `backend-request`

- `request-get-conn-latency`: time to get a connection from the connection pool
- `request-dns-latency`: time spen5 resolving a DNS name
- `request-tls-latency`: time spent on TLS Handshake

### Traces of the `backend` layer
**Stage Span** attributes (controlled by the `disable_stage` flag: will not appear if set to `true`):

- `krakend.stage`: always with value `proxy`
- `complete`: a `true` / `false` value to know when a response is complete (all
    backends returned a successful response).
- `canceled`: if it appears, it will always be `true` and indicates a request
    that has been canceled (usually when parallel requests are used).
- `error`: in case an error happens, the error description.

**read-tracer** span with when `read_payload` option is set to true.
