---
lastmod: 2023-03-16
old_version: true
date: 2018-04-05
linktitle: RabbitMQ Producer
title: AMQP Producer Integration in the API Gateway (RabbitMQ)
description: Integrate AMQP producers with KrakenD for seamless communication with backend systems. Follow our documentation to set up reliable and scalable messaging with Advanced Message Queuing Protocol (AMQP).
weight: 110
menu:
  community_v2.8:
    parent: "050 Non-REST Connectivity"
meta:
  since: v0.9
  source: https://github.com/krakend/krakend-amqp
  namespace:
  - backend/amqp/producer
  scope:
  - backend
  log_prefix:
  - "[BACKEND: /foo][AMQP]"
---

The AMQP producer component allows to **send messages to a queue** through the API Gateway.

The configuration of the queue is a straightforward process. To connect the endpoints to the messaging system you only need to include the `extra_config` key with the namespace `backend/amqp/producer`.

The parameters of this integration follow the AMQP specification. To understand
what are the implications of a certain parameter, see the **[AMQP Complete Reference Guide](https://www.rabbitmq.com/amqp-0-9-1-reference.html)**.

**KrakenD creates both the exchange and the queue for you**.


## Producer Configuration

```json
{
    "backend": [{
        "host": ["amqp://guest:guest@myqueue.host.com:5672"],
        "disable_host_sanitize": true,
        "extra_config": {
            "backend/amqp/producer": {
                "name": "queue-1",
                "exchange": "some-exchange",
                "durable": true,
                "delete": false,
                "no_wait": true,
                "no_local": false,
                "routing_key": "#",
                "mandatory": false,
                "immediate": false,
                "backoff_strategy": "exponential-jitter"
            }
        }
    }]
}
```

{{< schema version="v2.8" data="backend/amqp/producer.json" >}}

Additionally, the items below are parameter keys that can be present in the endpoint URL and are passed to the producer. Parameters need **capitalization on the first letter**.

{{< note title="Parameters' first character uppercased" >}}
Notice the **capitalization** of the first letter of the parameter names at the configuration below used in `exp_key`, `reply_to_key`, `msg_id_key`, `priority_key`, and `routing_key`.
{{< /note >}}

For instance, an `endpoint` URL could be declared as `/produce/{a}/{b}/{id}/{prio}/{route}` and the producer knows how to map them with a configuration like this:

```json
{
  "backend/amqp/producer": {
    "@comment": "Rest of the settings omitted",
    "exp_key":"A",
    "reply_to_key":"B",
    "msg_id_key":"Id",
    "priority_key":"Prio",
    "routing_key":"Route"
  }
}
```

## Connection retries
The gateway will try to reconnect to the AMQP server if the connection is lost for any reason. You can set the [back-off strategy](/docs/v2.8/async/#backoff-strategies) that better fits your needs.