---
lastmod: 2023-03-16
old_version: true
date: 2018-04-05
linktitle: RabbitMQ Consumer
title: Gateway integration with RabbitMQ consumers
weight: 90
menu:
  community_v2.3:
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

To create Async agents that consume messages asynchronously and without requiring a user request, see [Async Agents](/docs/v2.3/async/).

The parameters of this integration follow the AMQP specification. To understand
what are the implications of a certain parameter, see the **[AMQP Complete Reference Guide](https://www.rabbitmq.com/amqp-0-9-1-reference.html)**.

**KrakenD creates both the exchange and the queue for you**.


## Configuration
The consumer retrieves messages from the queue when a KrakenD endpoint plugs to its AMQP backend. The recommendation is to connect consumers to `GET` endpoints.

A single endpoint can consume messages from N queues, or can consume N messages from the same queue by adding N backends with the proper queue name.

See [Async Agents](/docs/v2.3/async/) to consume messages without an endpoint.

The needed configuration to run a consumer is:

```json
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
                "auto_ack": false,
                "backoff_strategy": "exponential-jitter"
            }
        }
    }]
}
```

{{< schema version="v2.3" data="backend/amqp/consumer.json" >}}

## Connection retries
The gateway will try to reconnect to the AMQP server if the connection is lost for any reason. You can set the [back-off strategy](/docs/v2.3/async/#backoff-strategies) that better fits your needs.
