---
lastmod: 2022-01-21
date: 2022-01-21
linktitle:  AMQP driver for Async Agent
title: AMQP driver for Async Agent
weight: 20
images:
  - /images/documentation/async-agents.png
skip_header_image: true
menu:
  community_current:
    parent: "060 Event Driven Gateway"
meta:
  since: 2.0
  source: https://github.com/luraproject/lura
  scope:
  - async_agent.extra_config
  log_prefix:
  - "[ASYNC: AgentName][AMQP]"
---
The AMQP driver for **Async agents** allows you to have KrakenD consuming AMQP queues autonomously. Routines listening to AMQP queues will react by themselves to new events and push data to your backends.

This driver is different from the [AMQP backend consumer](/docs/backends/amqp-consumer/). As opposed to endpoints, async agents do not require users to request something to trigger an action. Instead, the agents connect to the queue and fire an action when an event is delivered.

## Async/AMQP Driver Configuration
The AMQP driver has to be placed inside the `extra_config` of the [async component](/docs/async/agent/) and allows you connect to an AMQP queue (e.g: RabbitMQ). The settings are as follows:

{{< highlight json >}}
{
    "async/amqp": {
        "host": "amqp://guest:guest@localhost:5672/",
        "name": "krakend",
        "exchange": "foo",
        "durable": true,
        "delete": false,
        "exclusive": false,
        "no_wait": true,
        "prefetch_count": 5,
        "auto_ack": false,
        "no_local": true
    }
}
{{< /highlight >}}


- `host`- *string* The connection string, **ends in slash**. E.g: `amqp://user:password@host:5672/`
- `name` - *string* The queue name
- `exchange` - *string* The entity name where messages are retrieved (must have a **topic** type if already exists).
- `durable` - *bool* `true` is recommended, but depends on the use case. Durable queues will survive server restarts and remain when there are no remaining consumers or bindings.
- `delete` - *bool* `false` is recommended to avoid deletions when the consumer is disconnected
- `exclusive` - *bool* `true` if only this consumer can access the queue
- `no_wait` - *bool*: When true, do not wait for the server to confirm the request and immediately begin deliveries. If it is not possible to consume, a channel exception will be raised, and the channel will be closed.
- `prefetch_count` - *int* (optional): The number of messages you want to prefetch before consuming them.
- `prefetch_size` - *int* (optional): The number of bytes you want to use to prefetch messages.
- `no_local` - *bool* (optional) - The no_local flag is not supported by RabbitMQ.
- `auto_ack` - *bool*. When KrakenD retrieves the messages, regardless of the success or failure of the operation, it marks them as acknowledged.