---
lastmod: 2025-02-06
old_version: true
date: 2022-01-28
linktitle: Check plugin dependencies
title: "Checking dependencies of plugins"
description: Learn how to check your custom KrakenD plugins with the check-plugin command and ensure that your developments are compatible and loadable by KrakenD during runtime.
weight: 30
notoc: false
meta:
    since: 2.0
menu:
  community_v2.10:
    parent: "180 Extending with custom code"
---
The Go plugin system requires you to compile the main application and its plugins using the same ecosystem. This means that KrakenD and your plugins must use the same Go version, the same version of any **imported libraries**, the same system architecture, and the same GLIBC/MUSL libraries. Therefore, knowing in advance that you are using libraries that are incompatible with KrakenD when [writing custom plugins](/docs/v2.10/extending/writing-plugins/) is key.

The `{{< product check_plugin_command >}}` command helps you validate the part of the **dependencies used by your plugins**, which will determine whether the plugin is compatible. Go programs define their dependencies in their `go.sum` file, and this is all you need to check the compatibility.

## Usage of the check command
The command compares your plugin's `go.sum` file with the libraries initially used to compile the running binary. A detailed list will be shown if there are any incompatibilities between your plugin and KrakenD.

If you integrate this command as part of your CI/CD pipeline or `Dockerfile` build, it will exit with a status code `0` when your plugin's libraries are compatible with KrakenD and with a status code `1` when they are not.

The `{{< product check_plugin_command >}}` command accepts the following options:

{{< terminal title="Usage of KrakenD check" >}}
{{< product check_plugin_command >}} -h

{{< ascii-logo >}}

Version: 2.10

Checks your plugin dependencies are compatible and proposes commands to update your dependencies.

Usage:
  krakend 2.10 [flags]

Examples:
 krakend 2.10 -g 1.19.0 -s ./go.sum -f

Flags:
  -f, --format        Shows fix commands to update your dependencies
  -g, --go string     The version of the go compiler used for your plugin (default "1.22.11")
  -h, --help          help for check-plugin
  -l, --libc string   Version of the libc library used
  -s, --sum string    Path to the go.sum file to analyze (default "./go.sum")
{{< /terminal >}}

## Flags
Use `{{< product check_plugin_command >}}` in combination with the following flags:

- `-f` or `--format` to let KrakenD suggest you about the `go get` commands you should launch.
- `-s` or `--sum` to specify the path to the `go.sum` file of your plugin.
- `-g` or `--go` to specify the Go version you are using to compile the plugin
- `-l` or `--libc` to specify the libc version installed in the system. The libc version must have the prefix `MUSL-`, `GLIBC-`, and `DARWIN-`. For instance, a plugin in Mac Monterrey might use `DARWIN-12.2.1`, an Alpine container will need something like `MUSL-1.2.2`, and a Linux box will have `GLIBC-2.32`. To know your glibc version execute the [Find GLIBC script](https://github.com/krakend/krakend-ce/blob/master/find_glibc.sh). When there are incompatibilities because the operating system is different, but the libraries (glibc or musl) are in the **exact same version**, it is safe to ignore them.

The example below shows an example of a plugin that uses several libraries that are incompatible with KrakenD:

{{< terminal title="Checking a failing plugin example" >}}
{{< product check_plugin_command >}} --go 1.17.7 --libc MUSL-1.2.2 --sum ../plugin-tools/go.sum
15 incompatibility(ies) found...
go
    have: 1.17.0
    want: 1.16.4
libc
    have: MUSL-1.2.2
    want: GLIBC-2.31
github.com/gin-gonic/gin
    have: v1.6.3
    want: v1.7.7
github.com/go-playground/locales
    have: v0.13.0
    want: v0.14.0
github.com/go-playground/universal-translator
    have: v0.17.0
    want: v0.18.0
github.com/go-playground/validator/v10
    have: v10.2.0
    want: v10.9.0
github.com/golang/protobuf
    have: v1.3.3
    want: v1.5.2
github.com/json-iterator/go
    have: v1.1.9
    want: v1.1.12
github.com/leodido/go-urn
    have: v1.2.0
    want: v1.2.1
github.com/mattn/go-isatty
    have: v0.0.12
    want: v0.0.14
github.com/modern-go/concurrent
    have: v0.0.0-20180228061459-e0a39a4cb421
    want: v0.0.0-20180306012644-bacd9c7ef1dd
github.com/modern-go/reflect2
    have: v0.0.0-20180701023420-4b7aa43c6742
    want: v1.0.2
github.com/ugorji/go/codec
    have: v1.1.7
    want: v1.2.6
golang.org/x/sys
    have: v0.0.0-20200116001909-b77594299b42
    want: v0.0.0-20211004093028-2c5d950f24ef
golang.org/x/text
    have: v0.3.2
    want: v0.3.7
{{< /terminal >}}

## Fixing plugin dependencies
A quick attempt to fix your dependencies is to run the command with the `-f` flag, which will suggest a series of `go get` commands that you can execute to solve the incompatibilities. For instance:

{{< terminal title="Fixing dependencies" >}}
{{< product check_plugin_command >}} -s ~/Downloads/go.sum -f
12 incompatibility(ies) found...
go get cloud.google.com/go/pubsub@v1.19.0
go get github.com/census-instrumentation/opencensus-proto@v0.3.0
go get github.com/google/martian@v2.1.1-0.20190517191504-25dcb96d9e51+incompatible
go get github.com/googleapis/gax-go/v2@v2.2.0
go get github.com/hashicorp/golang-lru@v0.5.4
go get golang.org/x/mod@v0.6.0-dev.0.20211013180041-c96bc1413d57
go get golang.org/x/oauth2@v0.0.0-20220309155454-6242fa91716a
go get golang.org/x/sys@v0.0.0-20220330033206-e17cdc41300f
go get golang.org/x/time@v0.0.0-20220224211638-0e9765cccd65
go get google.golang.org/api@v0.74.0
go get google.golang.org/genproto@v0.0.0-20220502173005-c8bf987b8c21
go get google.golang.org/grpc@v1.46.0
{{< /terminal >}}

Copy and paste your terminal's `go get` commands to update the dependencies. The commands are in alphabetical order.

You might need to use the `-f` several times and use `go mod tidy` as well.
