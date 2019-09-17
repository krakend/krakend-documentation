---
lastmod: 2019-01-15
date: 2019-01-14
toc: true
linktitle: Writing plugins
title: Writing custom plugins
weight: 10
menu:
  documentation:
    parent: extending
images:
- /images/documentation/krakend-plugins.png
---

KrakenD's modular design allows you to extend its functionality by adding your custom code. As an engineer, it's always tempting to start writing code but **the majority of scenarios do not require writing any**. Existing modules, middleware, and plugins suffice almost everyone's needs.

It's important to remark, that if you want to do complex business logic checks and transformations, aside from the core functionality, there is specific scripting designed for that without compiling Go code. Examples are the [expression language](/docs/endpoints/common-expression-language-cel/), [Martian transformations](/docs/endpoints/martian/), or [Lua scripting](/docs/endpoints/lua/).

Let's get you started building custom code!

# Where to start customizing

The word plugin appears in many places over the Internet, but when we talk about plugins we refer to **[Go plugins](https://golang.org/pkg/plugin/)**.

`middleware != plugin`

KrakenD API Gateway is a composition of a framework and many other pieces and repositories that compile in a single, final, binary. We refer to these pieces as **middleware**, **components**, **modules**, or **packages** `¯\_(ツ)_/¯`.

**A plugin is a parallel program** (and a different binary) that when running in conjunction with KrakenD can participate in the processing.

Plugins and middlewares are close concepts but do not confuse them. The middleware compiles inside the KrakenD binary while plugins compile in **another binary**.

# Approaches to extend KrakenD

Either you write plugins, or you write middleware. These are the three options:

1.  Write and inject a plugin in the router layer
2.  Write and inject a plugin in the proxy layer
3.  Write a brand new middleware and compile KrakenD with it

Choosing one approach over another depends on what you want to accomplish. A simple orientation can be:

> Do you want to modify the request of the user before KrakenD starts processing?
> Go for the router plugin.
> Do you want to change how KrakenD interacts with your backend services?
> Go for the proxy plugin
> Do you want to change the internals of the pipes, add tooling, integrations, etc.?
> Write your custom middleware

# Sequence of requests and responses

A recommended read before going any further is "[Understanding the big picture](/docs/extending/the-big-picture/#the-important-packages)", and specially identify the important packages.

In a nutshell, the sequence of a request-response is as follows:

1.  The end-user sends an HTTP request to KrakenD.
2.  The `router` **transforms** the HTTP request into several `proxy` requests -HTTP or not- through a handler function.
3.  The `proxy` pipe fetches the data for all the requests, manipulates, aggregates... and returns the context to the `router`.
4.  The `router` converts back the proxy response into an HTTP response.

## Writing and injecting plugins

The sequence described above can be seen in the following diagram. Notice the two blue spots:

*   The http handler (router)
*   The http client (proxy)

<img src="/images/documentation/krakend-plugins.png" class="img-fluid text-center px-5 py-2">

The blue spots indicate the places where you can register your custom plugins.

### Writing proxy plugins (http client)

**HTTP client** plugins execute in the proxy layer. They allow you to intercept, transform, and manipulate the requests before they hit your backend services.

Writing an HTTP client requires to implement the [plugin client interface](https://github.com/devopsfaith/krakend/tree/master/transport/http/client/plugin). After doing this, your plugin registers itself on KrakenD during startup time.

A complete and working example of this type of plugin can be found in the article [gRPC-gateway as a KrakenD plugin](/blog/krakend-grpc-gateway-plugin/), which builds a gRPC-gateway to connect to your backends. Go through the article and linked sources to get your plugin working.

### Writing router plugins (http handler)

The router layer receives the user requests. You can intercept the request before KrakenD gets it to do any transformation or operation you want.

Writing an http handler plugin requires you to implement the [plugin server interface](https://github.com/devopsfaith/krakend/tree/master/transport/http/server/plugin).

### Writing your custom middleware

The last option is to write code and compile it along with KrakenD. When writing your custom code, the usual choice is to fork the [KrakenD-CE](https://github.com/devopsfaith/krakend-ce) repository.

The KrakenD-CE repository is the one assembling all the blocks and manages the dependencies (including the [KrakenD Framework](https://github.com/devopsfaith/krakend)), and lets you effortlessly include your company customizations.

The small drawback of this approach is that you need to maintain your custom version, which differs from our official binaries.

There are many examples of different modules, included in KrakenD-CE and not on our [contributions list](https://github.com/devopsfaith/krakend-contrib).

A relaxed start to build a component KrakenD is our article "[Website development as a sysadmin"](/blog/website-development-as-a-sysadmin/) where you can find custom code to add automatic API authentication against a backend.

# Ask the community

Finally, we recommend you joining our [Slack channel](/support) and explain what you are trying to do. Probably there is someone who was in your situation before, and you might even get a free code snippet!
