---
lastmod: 2023-02-02
date: 2019-01-14
linktitle: Building custom plugins
title: Building custom plugins for KrakenD API Gateway
description: Learn how to extend the functionality of KrakenD API Gateway by writing and building custom plugins, enabling custom business logic and workflows
weight: 10
skip_header_image: true
menu:
  community_current:
    parent: "180 Extending with custom code"
images:
- /images/documentation/krakend-plugins.png
---
All different types of plugins let you freely implement your logic without restrictions. However, make sure to write them implementing the correct interface and compile them respecting the requirements. In this document, we will see how to do it right.

{{< note title="Introduction to plugins" type="info" >}}
Before getting your hands dirty, read the [introduction to plugins](/docs/extending/) to understand **the different plugins** you can use and choose the one that best adapts to your needs.
{{< /note >}}


## Plugin requirements
Writing plugins isn't complicated per se, but Go is very strict with the environment where you compile and load them. Therefore, the following principles are essential:

- **Right interface**: Your plugin must implement the proper interface (see each plugin type).
- **Same Go version**: Your plugin and KrakenD are compiled with the same Go version. E.g., you cannot build a plugin on Go 1.19 and load it on a KrakenD assembled with Go 1.18.
- **Same architecture/platform**: KrakenD and your plugin have been compiled in the same architecture. E.g., you cannot compile a plugin in a Mac and use it in a Docker container.
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

The command will generate a `yourplugin.so` file (name it as you please) that you can now copy into a `{{< product image >}}:{{< product latest_version >}}` Docker image (but not to tag mismatching the builder), and load it as described in [injecting plugins](/docs/extending/injecting-plugins/).

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

{{< terminal title="Build your plugin for Alpine ARM64" >}}
docker run -it -v "$PWD:/app" -w /app \
-e "CGO_ENABLED=1" \
-e "CC=aarch64-linux-musl-gcc" \
-e "GOARCH=arm64" \
-e "GOHOSTARCH=amd64" \
{{< product image_plugin_builder >}}:{{< product latest_version >}}-linux-generic \
go build -ldflags='-extldflags=-fuse-ld=bfd -extld=aarch64-linux-musl-gcc' \
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