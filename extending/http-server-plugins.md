---
lastmod: 2021-05-21
date: 2021-05-21
toc: true
linktitle: HTTP server plugins
title: HTTP handler plugins
weight: 10
notoc: true
menu:
  community_current:
    parent: "150 Custom Plugins and Middleware"
meta:
  since: 2.0
  namespace:
  - plugin/http-server
images:
- /images/documentation/krakend-plugins.png
- /images/documentation/http-handler-plugin.png
---
The HTTP handler plugins (or HTTP server plugins) belong to the **router layer** and let you modify the requests before KrakenD starts processing them, or modify the final response back to the user. The HTTP handler plugin allows you to write your own server right in KrakenD and lets you implement anything you can imagine. This plugin type is so powerful that can be used to implement custom monetization, tracking, tenant control, special protocol conversion, and heavy modifications to put a few examples.

{{< note title="What is a Handler?" >}}A Handler responds to an HTTP request, and is an interface modeling an HTTP processor.{{< /note >}}

From KrakenD's perspective, **your handler plugins are black boxes** that expose an `http.Handler`, and you can do anything you want inside. Each plugin is wrapping the next element in the pipe, meaning that for some operations, **it must deal with an HTTP request and response writer**. If you chain several plugins you will add **two extra cycles** of decoding and encoding the body:

![http handler plugin](/images/documentation/http-handler-plugin.png)

## HTTP handler interface
{{< note title="Writing plugins" type="tip" >}}
Read the introduction to [writing plugins](/docs/extending/writing-plugins/) for compilation and configuration options if you haven't done it yet.
{{< /note >}}

To start with a *hello world* for your first plugin you have to implement the plugin server interface by copying the example provided in the Go documentation:

{{< button-group >}}
{{< button url="https://godoc.org/github.com/luraproject/lura/transport/http/server/plugin" text="Go Documentation" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253" />
</svg>{{< /button >}}
{{< /button-group >}}
