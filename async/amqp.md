---
lastmod: 2022-01-21
date: 2022-01-21
linktitle:  AMQP driver for Async Agent
title: AMQP driver for the Asynchronous Agent
description: Leverage Asynchronous AMQP Agents in KrakenD for high-performance messaging and event-driven architectures
weight: 510
images:
  - /images/documentation/async-agents.png
skip_header_image: true
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
meta:
  since: 2.0
  source: https://github.com/luraproject/lura
  scope:
  - async_agent
  log_prefix:
  - "[ASYNC: AgentName][AMQP]"
---
The AMQP driver for **Async agents** allows you to have KrakenD consuming AMQP queues autonomously. Routines listening to AMQP queues will react by themselves to new events and push data to your backends.

This driver is different from the [AMQP backend consumer](/docs/backends/amqp-consumer/). As opposed to endpoints, async agents do not require users to request something to trigger an action. Instead, the agents connect to the queue and fire an action when an event is delivered.

## Async/AMQP Driver Configuration
The AMQP driver has to be placed inside the `extra_config` of the [async component](/docs/async/) and allows you connect to an AMQP queue (e.g: RabbitMQ). The settings are as follows:

```json
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
```

{{< schema data="async/amqp.json" >}}
