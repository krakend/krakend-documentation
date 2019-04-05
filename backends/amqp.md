---
lastmod: 2018-04-05
date: 2018-04-05
linktitle: AMQP - RabbitMQ
title: API Gateway integration with RabbitMQ
weight: 90
since: 0.9
source: https://github.com/devopsfaith/krakend-amqp
menu:
  documentation:
    parent: backends
---
When an AMQP queue such as RabbitMQ wants to be used as a backend for the API Gateway, the AMQP component allows you directly plug the gateway with your AMQP hosts.

The configuration of the queue is a straightforward process. You need to include the `extra_config` key with the namespace `github.com/devopsfaith/krakend-amqp`. After configuring the queue your endpoints are connected to the queues.

The following sample configuration demonstrates the different options you can use:

		"backend": [
			{
				"host": [
					"amqp://guest:guest@myqueue.host.com:5672"
				],
				"extra_config": {
					"github.com/devopsfaith/krakend-amqp": {
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