---
lastmod: 2021-05-21
date: 2021-05-21
toc: true
linktitle: HTTP client plugins
title: HTTP Client plugins (proxy layer)
weight: 20
menu:
  community_current:
    parent: "150 Custom Plugins and Middleware"
meta:
  since: 2.0
  namespace: 
  - plugin/http-client
images:
- /images/documentation/krakend-plugins.png
- /images/documentation/http-client-plugin.png
---
The **HTTP client** plugins execute in the proxy layer, this is when KrakenD tries to reach your backends for content. They allow you to intercept, transform, and manipulate the requests **before they hit your backend services**, and its way back. It is the perfect time to modify the request before it reaches the backend.

HTTP client plugins cannot be chained. You can use up to one plugin per backend connection.


![http handler plugin](/images/documentation/http-client-plugin.png)

## HTTP client interface
{{< note title="Writing plugins" type="tip" >}}
Read the introduction to [writing plugins](/docs/extending/writing-plugins/) for compilation and configuration options if you haven't done it yet.
{{< /note >}}

To start with a *hello world* for your first plugin you have to implement the plugin client interface by copying the example provided in the Go documentation:

{{< button-group >}}
{{< button url="https://godoc.org/github.com/luraproject/lura/transport/http/client/plugin" text="Go Documentation" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253" />
</svg>{{< /button >}}
{{< /button-group >}}

A more complete example of this type of plugin that the basic plugin above can be found in the article [gRPC-gateway as a KrakenD plugin](/blog/krakend-grpc-gateway-plugin/), which builds a gRPC-gateway to connect to your backends. Go through the article and linked sources to get your plugin working.
