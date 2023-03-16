---
lastmod: 2022-01-24
old_version: true
date: 2018-04-05
linktitle: RabbitMQ Producer
title: Gateway integration with RabbitMQ producers
weight: 91
menu:
  community_v2.2:
    parent: "050 Backends Configuration"
meta:
  since: 0.9
  source: https://github.com/krakendio/krakend-amqp
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
                "prefetch_count": 10,
                "prefetch_size": 1024,
                "mandatory": false,
                "immediate": false
            }
        }
    }]
}
```

- `name` - *string* as the queue name
- `exchange` - *string* the exchange name (must have a **topic** type if already exists).
- `routing_key` - *string*: The routing keys you will use to send messages.
- `durable` - *bool* `true` is recommended, but depends on the use case. Durable queues will survive server restarts and remain when there are no remaining consumers or bindings.
- `delete` - *bool* `false` is recommended to avoid deletions when the producer is disconnected
- `no_wait` - *bool*: When true, do not wait for the server to confirm the request and immediately begin deliveries. If it is not possible to consume, a channel exception will be raised and the channel will be closed.
- `prefetch_count` - *int* (optional): Is the number of messages you want to prefetch prior to consume them.
- `prefetch_size` - *int* (optional): Is the number of bytes you want to use to prefetch messages.
- `mandatory` - *bool* (optional): The exchange must have at least one queue bound when true. Defaults to `false`.
- `immediate` - *bool* (optional): A consumer must be connected to the queue when true. Defaults to `false`.

Additionally, the items below are parameter keys that can be present in the endpoint URL and are passed to the producer. Parameters need **capitalization on the first letter**.

- `exp_key` - *string*
- `reply_to_key` - *string*
- `msg_id_key` - *string*
- `priority_key` - *string* - Key of the request parameters that is used as the priority value for the produced message.
- `routing_key` - *string* - Key of the request parameters that is used as the routing value for the produced message.


{{< note title="Parameters' first character uppercased" >}}
Notice the capitalization of the first letter of the parameter names at the configuration below. For instance, when an endpoint parameter is defined as `{route}`, define it in the config as `Route`.
{{< /note >}}

For instance, an `endpoint` URL could be declared as `/produce/{a}/{b}/{id}/{prio}/{route}` and the producer knows how to map them with a configuration like this:

```json
{
    ...
    "exp_key":"A",
    "reply_to_key":"B",
    "msg_id_key":"Id",
    "priority_key":"Prio",
    "routing_key":"Route"
}
```
