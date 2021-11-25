---
lastmod: 2021-11-25
date: 2021-11-25
linktitle: Introduction to plugins
title: Introduction to custom plugins and middlewares
weight: -100
skip_header_image: true
menu:
  community_current:
    parent: "150 Custom Plugins and Middleware"
images:
- /images/documentation/krakend-plugins.png
---

**Plugins are soft-linked libraries**, thus a separated `.so` file, that when running in conjunction with KrakenD can participate in the processing. When we talk about plugins we refer to **[Go plugins](https://golang.org/pkg/plugin/)**.

**Middlewares** are the different repositories and components that when compiled all together they form the final KrakenD binary. If you want to change its behaviour you must recompile KrakenD.

{{< note title="Do I need a plugin?" type="question">}}
In most of the cases, you don't need a custom plugin. The combination of different functionalities offered by the middlewares might help you solve a myriad of scenarios, with special mention to [CEL](/docs/endpoints/common-expression-language-cel/), [Martian](/docs/backends/martian/), or even [Lua scripting](/docs/endpoints/lua/). If you'd like to introduce custom business logic, a plugin does not limit what you can do. Also, if you need a lot of performance, a Go plugin is much faster than a Lua script (generally speaking at least x10).
{{< /note >}}

Unlike KrakenD middlewares, plugins are **an independent binary** of your own and are not part of KrakenD itself. Plugins allow you to "*drag and drop*" custom functionality that interacts with KrakenD while still using the official binaries with no need to fork the code. 

Let's get you started building custom code!

## Types of plugins
There are four different types of plugins you can write:

1.  **[HTTP server plugins](/docs/extending/http-server-plugins/)** (or handler plugins): They belong to the **router layer** and let you modify the request before KrakenD starts processing it, or modify the final response. You can have several plugins at once.
2.  **[HTTP client plugins](/docs/extending/http-client-plugins/)** (or proxy client plugins): They belong to the **proxy layer** and let you change how KrakenD interacts (as a client) with your backend services. You can have one plugin per backend.
3.  **[Response Modifier plugins](/docs/extending/plugin-modifiers)**: They are strictly modifiers and let you change the responses received from your backends.
4.  **[Request Modififer plugins](/docs/extending/plugin-modifiers)**: They are strictly modifiers and let you change the requests sent to your backends.


All types of plugins are marked with colored dots in the following diagram. 

![Krakend Plugins](/images/documentation/krakend-plugins.png)

In a nutshell, **the sequence of a request-response** depicted in the graph of the plugins above is as follows:

1.  The end-user/server sends an HTTP request to KrakenD that is processed by the `router pipe`. One or more http handler plugins can be injected in this stage. 
2.  The `router pipe` **transforms** the HTTP request into one or several `proxy` requests -HTTP or not- through a handler function. The request modifier plugin can intercept this stage and make modifications.
3.  The `proxy pipe` fetches the data for all the requests through the selected transport layer. The http client plugin modifies the interaction with the backend. 
4.  The `proxy pipe` manipulates, aggregates, applies components... and returns the context to the `router pipe`. The response plugin can manipulate the data per backend or when everything is aggregated.
5.  The `router pipe` finally converts back the proxy response into an HTTP response.


## Custom middleware
The recommended way to customize KrakenD is through plugins. But as all open-source code, you can modify KrakenD and its middlewares and do your own version. When writing your custom code, fork the [KrakenD-CE](https://github.com/devopsfaith/krakend-ce) repository, and read "[Understanding the big picture](/docs/extending/the-big-picture/#the-important-packages)" to identify the important packages.


The **krakend-ce** repository is the one assembling all the middlewares and manages the dependencies (including [Lura](https://github.com/luraproject/lura)). It lets you effortlessly include your company customizations, as the project itself is mostly a wrapper for all components. Be aware that when you fork KrakenD you will need to maintain your custom version, which differs from the official binaries.

There are many examples of different modules (included in KrakenD-CE and not) on our [contributions list](https://github.com/devopsfaith/krakend-contrib). If create new middlewares, feel free to open a pull request and let us know.

A relaxed start to build a custom component for KrakenD is our article "[Website development as a sysadmin"](/blog/website-development-as-a-sysadmin/) where you can find custom code to add automatic API authentication against a backend. This functionality can be achieved without forking the code, but still, it is an illustrative example.