---
lastmod: 2022-01-24
date: 2018-04-05
linktitle: RabbitMQ Consumer
title: Gateway integration with RabbitMQ consumers
weight: 90
aliases: ["/docs/backends/amqp/"]
menu:
  community_current:
    parent: "050 Backends Configuration"
meta:
  since: 0.9
  source: https://github.com/krakendio/krakend-amqp
  namespace:
  - backend/amqp/consumer
  scope:
  - backend
  - async_agent
  log_prefix:
  - "[BACKEND: /foo][AMQP]"
---

The AMQP component allows to **send and receive messages to and from a queue** through the API Gateway.

The configuration of the queue is a straightforward process. To connect the endpoints to the messaging system you only need to include the `extra_config` key with the namespaces `backend/amqp/consumer` or `backend/amqp/producer`.

To create Async agents that consume messages asynchronously and without requiring a user request, see [Async Agents](/docs/async/).

The parameters of this integration follow the AMQP specification. To understand
what are the implications of a certain parameter, see the **[AMQP Complete Reference Guide](https://www.rabbitmq.com/amqp-0-9-1-reference.html)**.

**KrakenD creates both the exchange and the queue for you**.


## Configuration
The consumer retrieves messages from the queue when a KrakenD endpoint plugs to its AMQP backend. The recommendation is to connect consumers to `GET` endpoints.

A single endpoint can consume messages from N queues, or can consume N messages from the same queue by adding N backends with the proper queue name.

See [Async Agents](/docs/async/) to consume messages without an endpoint.

The needed configuration to run a consumer is:

{{< highlight json >}}
{
    "backend": [{
        "host": ["amqp://guest:guest@myqueue.host.com:5672"],
        "disable_host_sanitize": true,
        "extra_config": {
            "backend/amqp/consumer": {
                "name": "queue-1",
                "exchange": "some-exchange",
                "durable": true,
                "delete": false,
                "no_wait": true,
                "no_local": false,
                "routing_key": ["#"],
                "prefetch_count": 10,
                "prefetch_size": 1024,
                "auto_ack": false
            }
        }
    }]
}
{{< /highlight >}}

- `name` - *string* as the queue name
- `exchange` - *string* the exchange name (must have a **topic** type if already exists).
- `routing_key` - *list*: The list of routing keys you will use to consume messages.
- `durable` - *bool* `true` is recommended, but depends on the use case. Durable queues will survive server restarts and remain when there are no remaining consumers or bindings.
- `delete` - *bool* `false` is recommended to avoid deletions when the consumer is disconnected
- `no_wait` - *bool*: When true, do not wait for the server to confirm the request and immediately begin deliveries. If it is not possible to consume, a channel exception will be raised and the channel will be closed.
- `prefetch_count` - *int* (optional): Is the number of messages you want to prefetch prior to consume them.
- `prefetch_size` - *int* (optional): Is the number of bytes you want to use to prefetch messages.
- `no_local` - *bool* (optional) - The no_local flag is not supported by RabbitMQ.
- `auto_ack` - *bool*. When KrakenD retrieves the messages, regardless of the success or failure of the operation, it marks them as `ACK`. Defaults to `false`.
