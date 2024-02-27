---
lastmod: 2022-10-10
date: 2021-11-25
linktitle: Introduction to plugins
title: Extending KrakenD API Gateway - Plugin Development
description: Learn how to extend KrakenD API Gateway by developing custom plugins to add new functionalities and integrate with external systems
aliases: ["/docs/extending/introduction/"]
weight: -1
skip_header_image: true
menu:
  community_current:
    parent: "180 Extending with custom code"
images:
- /images/documentation/krakend-plugins.png
---

**Plugins are soft-linked libraries**, thus a separated `.so` file that, when running in conjunction with KrakenD, can participate in the processing. When we talk about plugins, we refer to **[Go plugins](https://golang.org/pkg/plugin/)**. You can create custom code, inject it into different parts of KrakenD processing, and still use the official KrakenD software without forking the code.

**Middlewares**, on the other hand, are the components that form the KrakenD binary when glued together along with the core engine. Suppose you want to change its behavior or add new ones, then you must recompile KrakenD. However, the architecture is designed in a way you will not need to change middleware, and coding middleware becomes unnecessary because the plugin system relieves you from that.

{{< note title="Do I need a plugin?" type="question">}}
In most cases, you don't need a custom plugin. The combination of different functionalities offered by the middlewares might help you solve a myriad of scenarios, with special mention to [CEL](/docs/endpoints/common-expression-language-cel/), [Martian](/docs/backends/martian/), or even [Lua scripting](/docs/endpoints/lua/). If you'd like to introduce custom business logic, a plugin does not limit what you can do. Also, if you need a lot of performance, a Go plugin is much faster than a Lua script (generally speaking, at least x10).
{{< /note >}}

Unlike KrakenD middlewares, plugins are **an independent binary** of your own and are not part of KrakenD itself. Plugins allow you to "*drag and drop*" custom functionality that interacts with KrakenD while still using the official binaries without needing to fork the code.

Let's get you started building custom code!

## Types of plugins
There are different types of plugins you can write, and most of the job is understanding what kind of plugin you are going to need. They are:

1.  **[HTTP server plugins](/docs/extending/http-server-plugins/)** (or handler plugins): They belong to the **router layer** and let you do **anything** as soon as the request hits KrakenD. For example, you can modify the request before KrakenD starts processing it, block traffic, make validations, change the final response, connect to third-party services, databases, or anything else you imagine, scary or not. You can also **stack several plugins at once**.
2.  **[Request/Response Modifier plugins](/docs/extending/plugin-modifiers/)**: They are strictly modifiers and let you change the request or response data to and from your backends. These are lighter than the rest.
3.  **[HTTP client plugins](/docs/extending/http-client-plugins/)** (or proxy client plugins): They belong to the **proxy layer** and let you change how KrakenD interacts (as a client) with a specific backend service. They are as powerful as server plugins, but their working influence is smaller. You can have **one plugin for the connecting backend call**, because client plugins are terminators.

All types of plugins are marked with colored pills in the following diagram.

![Diagram of plugin placement](/images/documentation/diagrams/plugin-types.mmd.svg)

In a nutshell, **the sequence of a request-response** depicted in the graph of the plugins above is as follows:

1. The end-user/server sends an HTTP request to KrakenD that is processed by the `router pipe`. One or more [HTTP server plugins](/docs/extending/http-server-plugins/) (a.k.a **http handlers**) can be injected in this stage.
2. The `router pipe` **transforms** the HTTP request into one or several `proxy` requests -HTTP or not- through a handler function. The [request modifier plugin](/docs/extending/plugin-modifiers/) can intercept this stage and make modifications, before or after the split/aggregation.
3. The `proxy pipe` fetches the data for all the requests through the selected transport layer. The [HTTP client plugin](/docs/extending/http-client-plugins/) modifies any interaction with the backend.
4. The `proxy pipe` manipulates, aggregates, applies components... and returns the context to the `router pipe`. The [response modifier plugin](/docs/extending/plugin-modifiers/)) can manipulate the data per backend or when everything is aggregated.
5. The `router pipe` finally converts back the proxy response into an HTTP response.

When you have chosen the type of plugin that best fits your scenario, it's time to [write your plugin](/docs/extending/writing-plugins/).

## Custom middleware
The recommended way to customize KrakenD is through plugins. But as with all open-source code, you can modify KrakenD and its middleware and do your version. When writing your custom code, fork the [KrakenD-CE](https://github.com/krakend/krakend-ce) repository, and read "[Understanding the big picture](/docs/design/#the-important-packages)" to identify the essential packages.


The **krakend-ce** repository is the one assembling all the middlewares and manages the dependencies (including [Lura](https://github.com/luraproject/lura)). It lets you effortlessly include your company customizations, as the project is mainly a wrapper for all components. Be aware that when you fork KrakenD, you must maintain your custom version, which differs from the official binaries.

There are many examples of different modules (included in KrakenD-CE and not) on our [contributions list](https://github.com/krakend/krakend-contrib). If you create a new middleware, feel free to open a pull request and let us know.

A relaxed start to build a custom component for KrakenD is our article "[Website development as a sysadmin"](/blog/website-development-as-a-sysadmin/), where you can find custom code to add automatic API authentication against a backend. Of course, you can achieve this functionality **without forking the code**, but still, it is an illustrative example.
