---
lastmod: 2024-05-11
date: 2024-05-11
linktitle: Introduction to Non-REST Connectivity
title: Non-REST Connectivity
description: Explore KrakenD's non-REST connectivity features, including RabbitMQ integration, GraphQL, Lambda functions, event-driven async agents, and AMQP support.
weight: -10
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
---

KrakenD supports a range of non-REST connectivity options that open the door to integration with message brokers, event-driven systems, and alternative protocols and communication styles. By providing built-in capabilities to interact with services beyond traditional REST APIs, KrakenD enables applications to handle complex, real-time data flows and asynchronous communication patterns.

You can connect to non-REST options using a REST interface and mix multiple services at once. For instance, imagine you have a REST endpoint `/subscribe`  targeted for a mobile application that inserts payload directly in a RabbitMQ when called. You could even make the gateway react to a new event on a queue (an Async Agent), regardless of the origin, and call a webhook inside your infrastructure with its payload.

These are two examples, but through the documentation, you will find a lot more of the non-REST connectivity features:

- **RabbitMQ Consumer and Producer**: KrakenD can both consume from and produce to RabbitMQ queues, supporting bidirectional messaging. When configured as a [RabbitMQ Consumer](/docs/backends/amqp-consumer/), KrakenD retrieves messages directly from RabbitMQ queues and processes them according to your routing configuration. As a [RabbitMQ Producer](/docs/backends/amqp-producer/), KrakenD sends messages to specified RabbitMQ queues, enabling it to trigger downstream processes or distribute messages asynchronously across systems.

- **Apache Kafka and other Publisher/Subscriber Patterns**: Aside from RabbitMQ, KrakenD supports [Pub/Sub communication with Apache Kafka](/docs/backends/pubsub/), AWS SNS, SQS, Azure Service Bus, GCP PubSub, or NATS.io. This pattern allows KrakenD to broadcast events to multiple systems without tightly coupling them, making it suitable for applications that require distributed, event-driven workflows.

- **GraphQL Support**: In addition to REST, KrakenD offers support for [GraphQL](/docs/backends/graphql/), allowing API clients to request precisely the data they need. With GraphQL, KrakenD is an efficient aggregator for complex data queries, providing clients with a flexible, schema-driven approach that can reduce over-fetching and under-fetching data.

- **Lambda Function Integration**: KrakenD can [invoke AWS Lambda functions directly](/docs/backends/lambda/), allowing it to act as a bridge between event sources and serverless compute resources. This integration enables applications to process data or execute business logic asynchronously without requiring dedicated infrastructure, a common pattern in serverless and event-driven architectures. With significant volumes, using KrakenD instead of the native AWS API Gateway can remarkably reduce your bill.

- **Event-Driven Async Agents**: KrakenD’s [async agents](/docs/async/) facilitate non-blocking, event-driven processing within the gateway. These agents allow KrakenD to handle events autonomously and asynchronously, making it ideal for workflows that require decoupled, reactive responses to changes in data or state. Async agents improve throughput in distributed systems by offloading tasks without waiting for synchronous HTTP responses.

In addition, the **Enterprise Edition**, includes the following options:

- **gRPC Client and Server**: KrakenD supports both gRPC client and server functionalities, enabling bidirectional gRPC communication. As a [gRPC client](/docs/enterprise/backends/grpc/), KrakenD can make requests to external gRPC services, allowing it to interact with gRPC-based APIs and fetch or send data as needed. When configured as a [gRPC server](/docs/enterprise/grpc/server/), KrakenD can receive and handle incoming gRPC requests, making it a versatile gateway for gRPC-based interactions. You can transparently convert REST to gRPC and vice versa if needed.

- **WebSockets**: KrakenD supports [WebSocket communication](/docs/enterprise/websockets/), facilitating real-time, bidirectional data exchange between clients and servers. This capability allows KrakenD to handle continuous data streams and is suitable for applications that require live updates or continuous interaction, such as chat applications, real-time notifications, and IoT device communication. You can use direct WebSockets or multiplexing, which really optimizes the infrastructure.

- **SOAP**: KrakenD can interface with [SOAP-based services](/docs/enterprise/backends/soap/), enabling the gateway to connect with legacy systems that rely on the XML-based SOAP protocol. You can choose then to expose the legacy services as JSON REST automatically, and even convert the methods to GET. Consumers of a SOAP legacy service with KrakenD can experience a modern API without knowing that the underlying data transfer is XML.

These non-REST connectivity features in KrakenD allow applications to manage complex data flows and event-based interactions beyond the limitations of HTTP, making KrakenD a versatile option for integrating diverse services within modern and legacy architectures.

Follow to the documents under this section to find out more about each one.
