---
lastmod: 2020-03-01
date: 2019-09-15
linktitle: Publisher/subscribe
title: Pub-Sub Backend Integration in KrakenD API Gateway
description: Integrate Pub-Sub backend into KrakenD API Gateway to enable event-driven communication and real-time data updates in your API ecosystem
weight: 130
images:
- /images/features-event-driven.png
menu:
  community_current:
    parent: "040 Routing and Forwarding"
meta:
  since: 1.0
  source: https://github.com/krakend/krakend-pubsub
  namespace:
  - backend/pubsub/publisher
  - backend/pubsub/subscriber
  scope:
  - backend
  log_prefix:
  - "[BACKEND: schema://host][PubSub]"
---
You can connect an endpoint to multiple publish/subscribe backends, helping you integrate with **event driven architectures**.

<!--more-->
For instance, a frontend client can push events to a queue using a REST interface. Or a client could consume a REST endpoint that is plugged to the last events pushed in a backend. You can even **validate messages and formats** as all the KrakenD available middleware can be used. The list of supported backend technologies is:

- AWS SNS (Simple Notification Service) and SQS (Simple Queueing Service)
- Azure Service Bus Topic and Subscription
- GCP PubSub
- NATS.io
- RabbitMQ
- Apache Kafka

## Configuration
To add pub/sub functionality to your backends include the namespaces `backend/pubsub/subscriber` and `backend/pubsub/publisher` under the `extra_config` of your `backend` section.

The `host` key defines the desired driver, and the actual host is usually set in an **environment variable** outside of KrakenD:

For a **subscriber**:

```json
{
	"host": ["schema://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
			"subscription_url": "url"
		}
	}
}
```

{{< schema data="backend/pubsub/subscriber.json" >}}


For a **publisher**:

```json
{
	"host": ["schema://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/publisher": {
			"topic_url": "url"
		}
	}
}
```

{{< schema data="backend/pubsub/publisher.json" >}}

See the specification of each individual technology.

**Example** (RabbitMQ):

Set the envvar `RABBIT_SERVER_URL='guest:guest@localhost:5672'` and add in the configuration:

```json
{
	"host": ["amqp://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
		"subscription_url": "myexchange"
		}
	}
}
```
## Supported PubSub Drivers


### GCP PubSub
[Google's Cloud Pub/Sub](https://cloud.google.com/pubsub/) is a fully-managed real-time messaging service that allows you to send and receive messages between independent applications.

The configuration you need to use is:

- `host`: `gcppubsub://`
- `url` for topics: `"projects/myproject/topics/mytopic"` or the shortened form `"myproject/mytopic"`
- `url` for subscriptions: `"projects/myproject/subscriptions/mysub"` or the shortened form `"myproject/mysub"`
- Environment variables:  `GOOGLE_APPLICATION_CREDENTIALS`, see [Google documentation](https://cloud.google.com/docs/authentication/production/).

Example:

```json
{
	"host": ["gcppubsub://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
		"subscription_url": "projects/myproject/subscriptions/mysub"
		}
	}
}
```


### NATS
[NATS.io](https://nats.io/) is a simple, secure and high performance open source messaging system for cloud native applications, IoT messaging, and microservices architectures.

Configuration:

- `host`: `nats://`
- Environment variable: `NATS_SERVER_URL`
- `url`: `mysubject`

No query parameters are supported.

Example:

```json
{
	"host": ["nats://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
			"subscription_url": "mysubject"
		}
	}
}
```


### AWS SNS
[Amazon Simple Notification Service (SNS)](https://aws.amazon.com/sns/) is a highly available, durable, secure, fully managed pub/sub messaging service that enables you to decouple microservices, distributed systems, and serverless applications. Amazon SNS provides topics for high-throughput, push-based, many-to-many messaging

AWS SNS sets the `url` without any `host` or environment variables, e.g:

```json
{
	"host": ["awssns:///arn:aws:sns:us-east-2:123456789012:mytopic"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
			"subscription_url": "?region=us-east-2"
		}
	}
}
```


### AWS SQS
[Amazon Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications.

AWS SQS sets the `url` without any `host` or environment variables, e.g:

Url: `awssqs://sqs-queue-url`

```json
{
	"host": ["awssqs://sqs.us-east-2.amazonaws.com/123456789012"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
			"subscription_url": "/myqueue?region=us-east-2"
		}
	}
}
```

### Azure Service Bus Topic and Subscription
[Microsoft Azure Service Bus](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions) supports a set of cloud-based, message-oriented middleware technologies including reliable message queuing and durable publish/subscribe messaging. These "brokered" messaging capabilities can be thought of as decoupled messaging features that support publish-subscribe, temporal decoupling, and load balancing scenarios using the Service Bus messaging workload.

Configuration:

- `host`: `azuresb://`
- Environment variable: `SERVICEBUS_CONNECTION_STRING`
- Topics: `mytopic`
- Subscriptions: `mytopic?subscription=mysubscription`

Note that for subscriptions, the subscription name must be provided in the `?subscription=` query parameter.

Example:

```json
{
	"host": ["azuresb://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
			"subscription_url": "mytopic"
		}
	}
}
```

### RabbitMQ
[RabbitMQ](https://www.rabbitmq.com/) is one of the most popular open source message brokers.

Rabbit can alternatively be configured using the [AMQP component](/docs/backends/amqp-consumer/).

Configuration:

- `host`: `rabbit://`
- Environment variable: `RABBIT_SERVER_URL`
- `url` for topics: `myexchange`
- `url` for subscriptions: `myqueue`

No query parameters are supported.

Example:

```json
{
	"host": ["rabbit://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
			"subscription_url": "myexchange"
		}
	}
}
```

### Kafka
[Apache Kafka](https://kafka.apache.org/) is a distributed streaming platform.

Kafka connection requires KrakenD >= `1.1`.

- `host`: `kafka://`
- Environment var: `KAFKA_BROKERS` pointing to the server(s), e.g: `KAFKA_BROKERS=192.168.1.100:9092`

**Kafka subscriptions**:

```json
{
	"host": ["kafka://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/subscriber": {
			"subscription_url": "group?topic=mytopic"
		}
	}
}
```

**Kafka topics**:
```json
{
	"host": ["kafka://"],
	"disable_host_sanitize": true,
	"extra_config": {
		"backend/pubsub/publisher": {
			"topic_url": "mytopic"
		}
	}
}
```
