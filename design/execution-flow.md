---
date: 2022-09-06
lastmod: 2022-09-06
linktitle: The execution flow
title: KrakenD Execution Flow
description: How KrakenD components are loaded and executed
menu:
  community_current:
    parent: "170 Design principles"
images:
- /images/documentation/diagrams/components-sequence-simple.mmd.png
skip_header_image: true
---
To truly master KrakenD, you should get familiar with **the concept of "pipes"** and how these pipes define the execution flow of KrakenD from request to response.

A recurring question we hear from developers:  ***Is the configuration X executed before the configuration Y?***. If you are unfamiliar with KrakenD, it's hard to tell, as **the declaration order does not matter** (with a few sequential exceptions). Instead, each piece acts in a specific part(s) of the request and response journey.

## What is a pipe?
**A pipe is a function** that receives a request message, processes it, and produces the response message and an error. A pipe binds to a specific transport layer (e.g., HTTP, gRPC).

Each pipe loads the different factories of the components of KrakenD, what you could call its features. For instance, you might want to connect KrakenD to a Lambda function, a Kafka server or rate limit its interaction with your API, etc. All these features are part of the `Backend` pipe.

## Execution flow (pipes)
There are six differentiated parts on KrakenD pipes, as depicted in the following diagram:
![Execution flow](/images/documentation/diagrams/components-sequence-simple.mmd.png)

- `HTTP Adapter` or also known as `RunServer` is the entry conduit that receives the request from the user. In this phase, you can add your custom HTTP Server plugins and execute other components like Analytics or CORS.
- `Router` is the pipe part that identifies to which endpoint a request goes and allows to perform security checks before this happens. Both the `HTTP Adapter` and the `Router`are shared across all pipes.
- `Endpoint` enters for a specific endpoint (e.g.: `/foo`) and is a stage that allows you to do all kinds of validations and functionalities like validating tokens or modifying the request. One endpoint can connect sequentially or in parallel to many services (HTTP or not), and the endpoint defines all the places KrakenD will connect to. In this stage KrakenD converts the HTTP request into an internal KrakenD request (its domain).
- `Proxy` is the phase that splits the request into many connections (when necessary) and the other way around, merging the data from multiple places. Inside this pipe you can work with the data before it's sent to all your backends, or after it comes back from them.
- `Async agent` more than a pipe is a trigger that, without any request intervention, will execute a `Proxy` pipe without having an associated endpoint.
- `Backend` is when KrakenD connects to your services. It's the stage where you can do more operations as it manages the connections, drivers, limits, etc., going in and out of your upstream services.

Once the request reaches the upstream services, the response follows the same path back of the pipe until it gets to the user. The same parts of the pipe defined above will let you modify, validate, discard, etc., responses through each stage's components.

## Components flow
The following diagram is the same as the one shown above but includes the **primary components** (not all) involved in the different pipes. We understand this graph it is not for the faint-hearted, but it will help you answer the specific question of what is loaded before and after:

![Sequence of KrakenD components](/images/documentation/diagrams/components-sequence-master.mmd.png)
