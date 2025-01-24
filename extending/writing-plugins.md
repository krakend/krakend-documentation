---
lastmod: 2025-01-23
date: 2019-01-14
linktitle: Writing plugins
title: Writing and building custom plugins
description: Learn how to extend the functionality of KrakenD API Gateway by writing and building custom plugins, enabling custom business logic and workflows
weight: 10
skip_header_image: true
menu:
  community_current:
    parent: "180 Extending with custom code"
images:
- /images/documentation/krakend-plugins.png
---
**Plugins are soft-linked libraries**, thus a separated `.so` file that, when running in conjunction with KrakenD, can participate in the processing. When we talk about plugins, we refer to **[Go plugins](https://golang.org/pkg/plugin/)**. You can create custom code, inject it into different parts of KrakenD processing, and still use the official KrakenD software without forking the code.

{{< note title="Do I need a plugin?" type="question">}}
In most cases, you don't need a custom plugin. The combination of different functionalities offered by the built-in functionality might help you solve a myriad of scenarios, with special mention to [CEL](/docs/endpoints/common-expression-language-cel/), [Martian](/docs/backends/martian/), or even [Lua scripting](/docs/endpoints/lua/). If you'd like to introduce custom business logic, a plugin does not limit what you can do. Also, if you need a lot of performance, a Go plugin is much faster than a Lua script (generally speaking, at least x10).
{{< /note >}}

Plugins are **an independent binary** of your own and are not part of KrakenD itself. Plugins allow you to "*drag and drop*" (so to speak) custom functionality that interacts with KrakenD while still using the official binaries without needing to fork the code.

Let's get you started building custom code!

## Types of plugins
There are different types of plugins you can write, and most of the job is understanding what kind of plugin you are going to need. Look at the following diagram, the most striking colors are where you can inject plugins:

![Diagram of plugin placement](/images/documentation/diagrams/plugin-types.mmd.svg)

The types of plugins depicted are:

1.  {{< badge color="#6f00f0">}}server{{< /badge >}} **[HTTP server plugins](/docs/extending/http-server-plugins/)** (or handler plugins) belong to the **router layer** and let you do **anything** as soon as the request hits KrakenD and before the routing. For example, you can modify the request before KrakenD starts processing it, block traffic, make validations, change the final response, connect to third-party services, databases, or anything else you imagine, scary or not. You can also **stack several plugins at once**.
2.  {{< badge color="#0000ff" >}}req/resp{{< /badge >}} **[Request/Response Modifier plugins](/docs/extending/plugin-modifiers/)** are strictly **data modifiers** and let you change the request or response data to and from your backends. These are lighter than the rest.
3. {{< badge color="#e900b7" >}}client{{< /badge >}} **[HTTP client plugins](/docs/extending/http-client-plugins/)** (or proxy client plugins) belong to the **proxy layer** and let you change how KrakenD interacts (as a client) with a specific backend service. They are as powerful as server plugins, but their working influence is smaller. You can have **one plugin for the connecting backend call**, because client plugins are terminators. When you set a client plugin you are replacing the internal KrakenD client, which means that you can lose instrumentation and other features in it.

In a nutshell, **the sequence of a request-response** depicted in the graph of the plugins above is as follows:

1. The end-user/server sends an HTTP request to KrakenD that is processed by the `router pipe`. One or more [HTTP server plugins](/docs/extending/http-server-plugins/) (a.k.a **http handlers**) can be injected in this stage.
2. The `router pipe` **transforms** the HTTP request into one or several `proxy` requests -HTTP or not- through a handler function. The [request modifier plugin](/docs/extending/plugin-modifiers/) can intercept this stage and make modifications, before or after the split/aggregation.
3. The `proxy pipe` fetches the data for all the requests through the selected transport layer. The [HTTP client plugin](/docs/extending/http-client-plugins/) modifies any interaction with the backend.
4. The `proxy pipe` manipulates, aggregates, applies components... and returns the context to the `router pipe`. The [response modifier plugin](/docs/extending/plugin-modifiers/)) can manipulate the data per backend or when everything is aggregated.
5. The `router pipe` finally converts back the proxy response into an HTTP response.

All different types of plugins let you freely implement your logic without restrictions. However, make sure to write them implementing the correct interface and compile them respecting the requirements. 

## Plugin requirements
Writing plugins isn't complicated per se, but **Go is very strict** with the environment where you compile and load them. Therefore, the following principles are essential:

- **Right interface**: Your plugin must implement the proper interface (see each plugin type).
- **Same Go version**: Your plugin and KrakenD are compiled with the same Go version. E.g., you cannot build a plugin on Go 1.19 and load it on a KrakenD assembled with Go 1.22.
- **Same architecture/platform**: KrakenD and your plugin must have been compiled in the same architecture. E.g., you cannot compile a plugin in a Mac and use it in a Docker container (use the builder, see below)
- **Same shared library versions**: When using external libraries if for any reason KrakenD also uses them, they must include identical versions.
- **Injection in the configuration**: Besides coding and compiling your plugin, you must add it to the `krakend.json` configuration.

Yes, it sounds rigorous, but fortunately, some tools will tell you about this, so you don't have to lose time thinking much about this. Let's see them below.

## Tools to write your plugins
Once you have decided what type of plugin to write and started developing it, you need to ensure that your plugin uses the library versions compatible with KrakenD. You can use the following tools:

- The command `krakend version` gives you information about the Go and Glibc versions used during compilation.
- The [command `check-plugin`](/docs/extending/check-plugin/) analyzes your `go.sum` file and warns you about incompatibilities.
- The **Plugin Builder** is an environment with the versions you need (see below)
- The [command `test-plugin`](/docs/extending/test-plugin/) loads a compiled a plugin and tells you if it is loadable or not.

## Plugin Builder
{{< note title="Do not compile locally, use the builder" type="warning" >}}
Regardless where you want to use your plugin, compiling your plugins using the right builder is the way to go. The builder makes sure that the **system architecture and Go version** match the destination, making the plugin loadable.

If you choose to compile locally without the builder, you use a different architecture and underlying libc libraries that will make your plugin unusable.
{{< /note >}}

There are two builders you should use, depending on where you want to run the plugin:

| Architecture | Alpine (Docker) | Non-Docker (on-premises) |
|--------------|---------------------------------------------------------------------|-----------------------------------------------------------------------------------|
| [AMD64](/docs/extending/writing-plugins/#compile-plugins-for-amd64)        | `{{< product image_plugin_builder >}}:{{< product latest_version >}}` | `{{< product image_plugin_builder >}}:{{< product latest_version >}}-linux-generic` |
| [ARM64](/docs/extending/writing-plugins/#compile-plugins-for-arm64)        | `{{< product image_plugin_builder >}}:{{< product latest_version >}}` with cross-compile instructions | `{{< product image_plugin_builder >}}:{{< product latest_version >}}-linux-generic` with cross-compile instructions |

When using Docker to deploy your gateway, our official KrakenD container uses **[Alpine](https://hub.docker.com/_/alpine)** as the base image. Therefore, to use your custom plugins, they must compile using the Alpine builder.

## Compile plugins for AMD64
To build your plugin for **Docker targets**, you only need to execute the following command inside the folder where your plugin is:

{{< terminal title="Build your plugin for Alpine" >}}
docker run -it -v "$PWD:/app" -w /app {{< product image_plugin_builder >}}:{{< product latest_version >}} go build -buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

The command will generate a `yourplugin.so` file (name it as you please) that you can now copy into a `{{< product image >}}:{{< product latest_version >}}` Docker image, and load it as described in [injecting plugins](/docs/extending/injecting-plugins/). **Never use a tag mismatching the builder and KrakenD**. If you want to load the plugin in a KrakenD version `x.y.x` make sure to build it on a builder `x.y.z`. Using `.so` files that were compiled in a builder with a different version, will mostly fail.

To build the plugin for **on-premises installations** use the following command instead:

{{< terminal title="Build your plugin for Non-Docker" >}}
docker run -it -v "$PWD:/app" -w /app {{< product image_plugin_builder >}}:{{< product latest_version >}}-linux-generic go build -buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

## Compile plugins for ARM64
Regardless of the host architecture you use when running the Docker builder, the **default plugin architecture target is AMD64**. Therefore, if you want to test the plugin on **ARM64** (e.g., a Macintosh, Raspberry, etc.), you must cross-compile it. This is because the plugin builder is available for AMD64 only, as emulation does not work well on Go compilation.

To cross-compile a plugin for **Docker ARM64**, you need to add extra flags when compiling the plugin:


{{< terminal title="Build your plugin for Alpine ARM64" >}}
docker run -it -v "$PWD:/app" -w /app \
-e "CGO_ENABLED=1" \
-e "CC=aarch64-linux-musl-gcc" \
-e "GOARCH=arm64" \
-e "GOHOSTARCH=amd64" \
{{< product image_plugin_builder >}}:{{< product latest_version >}} \
go build -ldflags='-extldflags=-fuse-ld=bfd -extld=aarch64-linux-musl-gcc' \
-buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

And the same command, changing the builder, when you need **on-premises** plugins for ARM64:

{{< terminal title="Build your plugin for non-Alpine ARM64" >}}
docker run -it -v "$PWD:/app" -w /app \
-e "CGO_ENABLED=1" \
-e "CC=aarch64-linux-gnu-gcc" \
-e "GOARCH=arm64" \
-e "GOHOSTARCH=amd64" \
{{< product image_plugin_builder >}}:{{< product latest_version >}}-linux-generic \
go build -ldflags='-extldflags=-fuse-ld=bfd -extld=aarch64-linux-gnu-gcc' \
-buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

Remember that the resulting plugin will only work on **ARM64** and that you cannot reuse plugins from one platform into another.

## Check your dependencies
As your custom plugins need to match the Go and libraries versions used to build KrakenD, you have to guarantee your plugin is compatible by checking the `go.sum` file with the command `check-plugin` ([read the documentation](/docs/extending/check-plugin/))

{{< terminal title="Checking plugins" >}}
krakend check-plugin -v 1.17.0 -s ../myplugin/go.sum
1 incompatibility(ies) found...
go
  have: 1.17.0
  want: 1.16.4
{{< /terminal >}}

Once you have written your plugin with the interface you have chosen, compile it in the same architecture type as follows:

{{< terminal title="Go compilation">}}
go mod init myplugin
go build -buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

Now [test it](/docs/extending/test-plugin/), or [load it](/docs/extending/injecting-plugins/) in KrakenD
