---
lastmod: 2019-01-15
old_version: true
date: 2019-01-14
toc: true
linktitle: The big picture
title: Design principles. The big picture.
description: Discover the KrakenD API architecture and its design principles, as well its most important internal packages.
weight: -1
menu:
  community_v2.9:
    parent: "310 Design principles"
images:
- /images/documentation/config-router-proxy-packages.png
skip_header_image: true
---
Before starting to dive into the KrakenD code, you should spend a few minutes understanding the big pieces of the system, how they work, and the philosophy behind it.

## The design rules
Let's start with the rules followed to code KrakenD (shared with [The Lura Project](https://luraproject.org)), as they answer architectural design questions:

* [Reactive is key](http://www.reactivemanifesto.org/)
* Failing fast is better than succeeding slow
* The simpler, the better
* Everything is pluggable
* Each request must be processed in its request-scoped context

## KrakenD internal states
When you start KrakenD, the system goes through two internal states: **building** and **working**. So let's see what happens in every state.

### Building state
The building state administers the service start-up and prepares the system before it can start receiving traffic. During the building state, three things happen:

- Parsing of the configuration to fix the system behavior
- Preparation of the middlewares (all components in KrakenD)
- Construction of the pipes (see below)

A `pipe` is a function that receives a request message, processes it, and produces the response message and an error. The KrakenD router binds the pipes to the selected transport layer (e.g., HTTP, gRPC).

When the building state finishes, the KrakenD service **will not need to calculate any route** or lookup for the associated handler function, as all the mapping is direct in-memory. This is a crucial difference with any other system and leads to a significant performance.

[Read more on pipes](/docs/v2.9/design/execution-flow/)

### Working state
The working state is when the system is ready and can process the requests. When they arrive, the `router` already has the request mapping with the handler function and triggers the pipe execution. The `proxy` is the step of the pipe that manipulates, aggregates, and does other data handling for the rest of the process.

As the handler functions are in the previous step, **KrakenD doesn't penalize the performance depending on the number of endpoints or the possible cardinality** of the URIs requested by the users.

## The important packages
The Lura Project (KrakenD's engine) is composed of a set of packages designed as building blocks for creating pipes and processors between an exposed endpoint and one or several API resources served by your backends.


![Lura packages](/images/documentation/config-router-proxy-packages.png)

The most important packages are:

1. the `config` package defines the service.
2. the `router` package sets up the endpoints exposed to the clients.
3. the `proxy` package adds the required middlewares and components for further processing of the requests and responses of the backends. It also manages the connections against those backends.

The rest of the framework packages contain some helpers and adapters for additional tasks, like encoding, logging, or service discovery.

Additionally, KrakenD bundles a lot of middleware and components that are in its scope and package. These packages and others are listed in our [KrakenD Contrib](https://github.com/krakend/krakend-contrib) repository.


### The `config` package

The `config` package contains the structs required for the service description.

The `ServiceConfig` struct defines the entire service. Initialize it before using it to ensure that all parameters are normalized and that default values are applied.

The `config` package also defines an interface for a file config parser and a parser based on the [Viper](https://github.com/spf13/viper) library.

### The `router` package

The `router` package contains an interface and several implementations for the KrakenD router layer using the `mux` router from the `net/http` and the `httprouter` wrapped in the `gin` framework.

The router layer is responsible for setting up the HTTP(S) services, binding the endpoints defined at the `ServiceConfig` struct, and transforming the HTTP request into proxy requests before delegating the task to the inner layer (proxy). Once the internal proxy layer returns a proxy response, the router layer converts it into a proper HTTP response and sends it to the user.

This layer can be easily extended to use any HTTP router, framework, or middleware of your choice.

### The `proxy` package

The `proxy` package is where most of the KrakenD components and features are. It defines two necessary interfaces designed to be stacked:

* *Proxy* is a function that converts a given context and request into a response.
* *Middleware* is a function that accepts one or more proxies and returns a single proxy wrapping them.

This layer transforms the request received from the outer layer (router) into a single or several requests to your backend services, processes the responses, and returns a single response.

Middlewares generate chained custom proxies depending on the workflow defined in the configuration until each possible branch ends in a transport-related proxy. These generated proxies can transform the input or clone it several times and pass it to the next element in the chain. Finally, they can modify the received response or responses, adding all kinds of features to the generated pipe.

The Lura Project provides a default implementation of the proxy stack factory.

#### Middlewares available

* The `balancing` middleware uses some strategy for selecting a backend host to query.
* The `concurrent` middleware improves the QoS by sending several concurrent requests to the next step of the chain and returning the first successful response using a timeout for canceling the generated workload.
* The `logging` middleware logs the received request and response and the segment execution duration.
* The `merging` middleware is a fork-and-join middleware. It is intended to split the request process into several concurrent processes, each against a different backend, and to merge all the received responses from those created pipes into a single one. It applies a timeout, as the `concurrent` one does.
* The `http` middleware completes the received proxy request by replacing the parameters extracted from the user request in the defined `URLPattern`.

#### Proxies available

* The `http` proxy translates a proxy request into an HTTP one, sends it to the backend API using an `HTTPClientFactory`, decodes the returned HTTP response with a `Decoder`, manipulates the response data with an `EntityFormatter`, and returns it to the caller.

#### Other components of the `proxy` package

The `proxy` package also defines the `EntityFormatter`, the block responsible for enabling a powerful and fast response manipulation.