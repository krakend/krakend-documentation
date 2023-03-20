---
lastmod: 2023-02-02
date: 2019-01-14
linktitle: Writing custom plugins
title: Writing custom plugins
weight: 1
skip_header_image: true
menu:
  community_current:
    parent: "150 Custom Plugins and Middleware"
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

## Plugin Builder
When using Docker to deploy your gateway, the official KrakenD container uses **[Alpine](https://hub.docker.com/_/alpine)** as the base image. Therefore, to use your custom plugins, they must compile within an Alpine container and use the same Go and Glibc versions as KrakenD. The **Plugin Builder** docker image spares you from this job.

{{< note title="Compiling plugins for on-premise" type="info" >}}
When you build plugins for **non-Docker targets**, instead of the Alpine image `{{< product image_plugin_builder >}}:{{< product latest_version >}}` you need to use the image `{{< product image_plugin_builder >}}:{{< product latest_version >}}-linux-generic`.
{{< /note >}}


You can get the plugin builder with the following:

{{< terminal title="Download plugin builder" >}}
docker pull {{< product image_plugin_builder >}}:{{< product latest_version >}}
{{< /terminal >}}

Then to build your plugin, you only need to execute the following command inside the folder where your plugin is:

{{< terminal title="Build your plugin" >}}
docker run -it -v "$PWD:/app" -w /app {{< product image_plugin_builder >}}:{{< product latest_version >}} go build -buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

The command will generate a `yourplugin.so` file (name it as you please) that you can now copy into a `{{< product image >}}:{{< product latest_version >}}` Docker image (but not to tag mismatching the builder), and load it as described in [injecting plugins](/docs/extending/injecting-plugins/).

{{< note title="Builds are for AMD64" type="warning" >}}
The builder will compile the plugin for `AMD64` by default. If you'd like to load the plugin on `ARM64` (e.g., Docker on Mac) see cross-compiling below.
{{< /note >}}


### Cross-compiling plugins (ARM64)
Regardless of the host architecture you use when running the Docker builder, the **plugin architecture target is AMD64**. Therefore, if you want to test the plugin on **ARM64** (e.g., a Macintosh, Raspberry, etc.), you must cross-compile it. This is because the plugin builder is available for AMD64 only, as emulation does not work well on Go compilation.

To cross-compile a plugin for ARM64, you need to add extra flags when compiling the plugin:

```bash
export CGO_ENABLED=1
export CC=aarch64-linux-musl-gcc
export GOARCH=arm64
export GOHOSTARCH=amd64
export EXTRA_LDFLAGS='-extldflags=-fuse-ld=bfd -extld=aarch64-linux-musl-gcc'
go build -ldflags="${EXTRA_LDFLAGS}" -buildmode=plugin -o yourplugin.so .
```
Remember that the resulting plugin will only work on **ARM64** and that you cannot reuse plugins from one platform into another.

## Compiling plugins without Docker
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

Now load it in KrakenD, as described in [injecting plugins](/docs/extending/injecting-plugins/)

## Debugging plugins in an IDE
If you'd like to debug your plugins with an IDE, you can enable **delve flags** when compiling. KrakenD must be compiled with these gcflags too. This option is not available on the Enterprise product.

{{< terminal title="Delve flags">}}
go build -gcflags='all=-N -l' -buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

Do not use these flags for the `.so` file you will use in production.