---
lastmod: 2018-04-05
date: 2018-04-05
linktitle: AMQP - RabbitMQ
title: API Gateway integration with AMQP messaging
weight: 90
since: 0.9
source: https://github.com/devopsfaith/krakend-amqp
menu:
  documentation:
    parent: backends
---
The AMQP component allows to **send and receive messages to and from a queue** through the API Gateway

The configuration of the queue is a straightforward process. To connect the endpoints to the messaging system you only need to include the `extra_config` key with the namespaces `github.com/devopsfaith/krakend-amqp/consume]` or `github.com/devopsfaith/krakend-amqp/produce]`.

The following configurations demonstrate both the **consumer** and the **producer** to create the whole publish/susbcribe pattern.

# Consumer
The consumer retrieves messages from the queue when a KrakenD endpoint is plugged to its AMQ backend. The needed configuration is as follows:

		"backend": [
			{
				"host": [
					"amqp://guest:guest@myqueue.host.com:5672"
				],
				"extra_config": {
					"github.com/devopsfaith/krakend-amqp/consume": {
						"name":           "queue-1",
						"exchange":       "some-exchange",
						"durable":        true,
						"delete":         false,
						"exclusive":      false,
						"no_wait":        true,
						"auto_ack":       false,
						"no_local":       false,
						"routing_key":    ["#"],
						"prefetch_count": 10
					}
				}
			}

# Producer
The producer publishes messages to the messaging system for your asynchronous consumption. The needed configuration is as follows:

		"backend": [
			{
				"host": [
					"amqp://guest:guest@myqueue.host.com:5672"
				],
				"extra_config": {
					"github.com/devopsfaith/krakend-amqp/produce": {
						"exchange":       "some-exchange",
						"durable":        true,
						"delete":         false,
						"exclusive":      false,
						"no_wait":        true,
						"mandatory": true,
						"immediate": false
					}
				}
			}