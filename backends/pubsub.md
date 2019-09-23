---
lastmod: 2019-09-15
date: 2019-09-15
linktitle: Publisher/subscribe
title: Using publisher/subscribe as backends
weight: 100
since: 1.0
source: https://github.com/devopsfaith/krakend-pubsub
images:
- /images/features-event-driven.png
menu:
  documentation:
    parent: backends
---
Since KrakenD 1.0 you can connect an endpoint to multiple publish/subscribe backends, helping you integrate with **event driven architectures**. For instance, a frontend client can push events to a queue using a REST interface. Or a client could consume a REST endpoint that us plugged to the last events pushed in a backend. You can even validate messages and formats, as all the KrakenD available middleware can be used. The list of supported backend technologies is:

- AWS SNS (Simple Notification Service) and SQS (Simple Queueing Service)
- Azure Service Bus Topic and Subscription
- GCP PubSub
- NATS.io
- RabbitMQ

# Configuration
To add pub/sub functionality to your backends include the following namespaces under the `extra_config` of your `backend`:

For a **subscriber**:

	"github.com/devopsfaith/krakend-pubsub/subscriber": {
		"subscription_url": "schema://url"
	}

For a **publisher**:

	"github.com/devopsfaith/krakend-pubsub/publisher": {
		"topic_url": "schema://url"
	}

The `schema://url` depends on the type of backend you want to connect, as described below:

## GCP PubSub
[Google's Cloud Pub/Sub](https://cloud.google.com/pubsub/) is a fully-managed real-time messaging service that allows you to send and receive messages between independent applications.

To connect to GCP PubSub the connection uses the default credentials from the environment, as described in [Google documentation](https://cloud.google.com/docs/authentication/production).

The schema you need to use is:

- Topics: `"gcppubsub://projects/myproject/topics/mytopic"` or the shortened form `"gcppubsub://myproject/mytopic"`
- Subscriptions: `"gcppubsub://projects/myproject/subscriptions/mysub"` or the shortened form `"gcppubsub://myproject/mysub"`

## NATS
[NATS.io](https://nats.io/) is a simple, secure and high performance open source messaging system for cloud native applications, IoT messaging, and microservices architectures.

Schema: `nats://mysubject`

The URL host+path is used as the subject. No query parameters are supported.


## AWS SNS
[Amazon Simple Notification Service (SNS)](https://aws.amazon.com/sns/) is a highly available, durable, secure, fully managed pub/sub messaging service that enables you to decouple microservices, distributed systems, and serverless applications. Amazon SNS provides topics for high-throughput, push-based, many-to-many messaging
Schema: `awssns:///sns-topic-arn`

For SNS topics, the URL's host+path is used as the topic Amazon Resource Name (ARN). Since ARNs have ":" in them, and ":" precedes a port in URL hostnames, leave the host blank and put the ARN in the path (e.g., "awssns:///arn:aws:service:region:accountid:resourceType/resourcePath").

## AWS SQS
[Amazon Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications.

Schema: `awssqs://sqs-queue-url`

For SQS topics and subscriptions, the URL's host+path is automatically prefixed with "https://" to create the queue URL.

## Azure Service Bus Topic and Subscription
[Microsoft Azure Service Bus](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions) supports a set of cloud-based, message-oriented middleware technologies including reliable message queuing and durable publish/subscribe messaging. These "brokered" messaging capabilities can be thought of as decoupled messaging features that support publish-subscribe, temporal decoupling, and load balancing scenarios using the Service Bus messaging workload.

Schema:

- Topics: `azuresb://mytopic`
- Subsciptions: `azuresb://mytopic?subscription=mysubscription`

The URL's host+path is used as the topic name. For subscriptions, the subscription name must be provided in the "subscription" query parameter.

## RabbitMQ
[RabbitMQ](https://www.rabbitmq.com/) is one of the most popular open source message brokers.

Schema:

- Topics: `rabbit://myexchange`
- Subsciptions: `rabbit://myqueue`

For topics, the URL's host+path is used as the exchange name. For subscriptions, the URL's host+path is used as the queue name. No query parameters are supported.