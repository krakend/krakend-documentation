---
lastmod: 2019-01-15
old_version: true
date: 2019-01-14
toc: true
linktitle: Writing custom plugins
title: Writing custom plugins
weight: 1
skip_header_image: true
menu:
  community_v2.0:
    parent: "150 Custom Plugins and Middleware"
images:
- /images/documentation/krakend-plugins.png
---
All different types of plugins let you freely implement your logic without restrictions. To start using your own plugins make sure to write them implementing the right interface and compile them respecting the requirements.

{{< note title="Introduction to plugins" type="info" >}}
Before getting your hands dirty, read the [introduction to plugins](/docs/v2.0/extending/) for understanding **the different types of plugins** you can use.
{{< /note >}}


## Plugin requirements
{{< note title="Plugin binaries are not cross-platform compatible" type="error" >}}
You must compile the plugin with the same architecture/platform where it will be run.
{{< /note >}}

Writing, compiling and using plugins need to comply with the following list:

- **Right interface**: Your plugin implements the proper interface (see each plugin type)
- **Same go version**: You compile the plugin using the same Go version KrakenD was compiled with
- **Same architecture/platform**: You compile the plugin using the same architecture where KrakenD will run. E.g., you cannot compile a plugin in a Mac and use it in a Docker container).
- **Same import versions**: When using external libraries if KrakenD also uses them, they must be in the same version.
- **Register and inject** your plugins in the configuration.


## Compiling plugins
As your custom plugins need to match the Go and libraries versions used to build KrakenD, you have to make sure your plugin is compatible by checking your `go.sum` file with the command `check-plugin` (read the documentation)

{{< terminal title="Term" >}}
krakend check-plugin -v 1.17.0 -s ../myplugin/go.sum
1 incompatibility(ies) found...
go
	have: 1.17.0
	want: 1.16.4
{{< /terminal >}}

Once your plugin is written with the plugin type interface you have chosen, compile it in the same architecture type as follows:

{{< terminal title="Go compilation">}}
go mod init yourmodulename
go build -buildmode=plugin -o yourplugin.so .
{{< /terminal >}}

### Compiling plugins inside Docker
The official KrakenD container uses **[Alpine](https://hub.docker.com/_/alpine)** as the base image. Therefore, when compiling your plugins inside a Docker that extends the official image with a go-alpine builder, you will need to install at least the following requirements in the `Dockerfile`:

{{< highlight Dockerfile >}}
RUN apk add make gcc musl-dev
{{< /highlight >}}

## Adding your custom plugin
Once you have developed your plugin is time to [inject it in the configuration](/docs/v2.0/extending/injecting-plugins/).