---
lastmod: 2022-08-22
old_version: true
date: 2022-01-28
linktitle: Checking your plugins
title: "Guide to check custom KrakenD plugins"
description: Learn how to check your custom KrakenD plugins with the check-plugin command and make sure that your developments are compatible and loadable by KrakenD during runtime.
weight: 20
notoc: true
meta:
    since: v2.0
menu:
  community_v2.5:
    parent: "180 Extending with custom code"
---
The `krakend check-plugin` command helps you validate the **compatibility of your custom plugins** that will run in conjunction with KrakenD.

The command compares your plugin's `go.sum` file with the libraries initially used to compile the running binary. If there are any incompatibilities between your plugin and KrakenD, it will show a detailed list.

If you integrate this command as part of your CI/CD pipeline, it will exit with a status code `0` when the libraries of your plugin are compatible with KrakenD and with a status code `1` when they are not.

Notice that the `check-plugin` command does not check the plugin's validity itself nor need its source code other than the `go.sum` file.

To get started writing your plugins see:

- [Introduction to custom plugins](/docs/v2.5/extending/)
- [Writing custom plugins](/docs/v2.5/extending/writing-plugins/)

The `krakend check-plugin` command accepts the following options:

{{< terminal title="Usage of KrakenD check" >}}
krakend check-plugin -h

{{< ascii-logo >}}

Version: 2.5

Validates that the active configuration file has a valid syntax to run the service.
Change the configuration file by using the --config flag

Usage:
  krakend check-plugin [flags]

Examples:
krakend check-plugin -g 1.17.0 -s ./go.sum

Flags:
  -f, --format        Dump the commands to update
  -g, --go string     The version of the go compiler used for your plugin (default "1.17.11")
  -h, --help          help for check-plugin
  -l, --libc string   Version of the libc library used
  -s, --sum string    Path to the go.sum file to analize (default "./go.sum")
{{< /terminal >}}

## Flags
Use `krakend check-plugin` in combination with the following flags:

- `-f` or `--format` to let KrakenD suggest you about the `go get` commands you should launch.
- `-s` or `--sum` to specify the path to the `go.sum` file of your plugin.
- `-g` or `--go` to specify the Go version you are using to compile the plugin.
- `-l` or `--libc` to specify the libc version installed in the system. The libc version must have the preffix `MUSL-`, `GLIBC-`, `DARWIN-`. For instance, a plugin in Mac Monterrey might use `DARWIN-12.2.1`, an Alpine container will need something like `MUSL-1.2.2`, and a Linux box will have `GLIBC-2.32`. To know your glibc version execute the [Find GLIBC script](https://github.com/krakend/krakend-ce/blob/master/find_glibc.sh)


{{< terminal title="Checking a failing plugin example" >}}
krakend check-plugin --go 1.17.7 --libc MUSL-1.2.2 --sum ../plugin-tools/go.sum
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

## Updating plugin dependencies
A quick attempt to fix your dependencies is to run the command with the `-f` flag, which will suggest a series of `go get` commands that you can execute to solve the incompatibilities. For instance:

{{< terminal title="Fixing dependencies" >}}
krakend check-plugin -s ~/Downloads/go.sum -f
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

Copy and paste the `go get` commands in your terminal to update the dependencies. The commands are in alphabetical order.

You might need to use the `-f` several times and use `go mod tidy` as well.