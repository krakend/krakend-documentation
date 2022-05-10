---
lastmod: 2022-01-28
date: 2022-01-28
aliases: ["/docs/extending/plugin-tools/"]
linktitle: Checking your plugins
title: Validate your plugin compatibility
description: KrakenD check-plugin Command
weight: 50
notoc: true
meta:
    since: 2.0
menu:
  community_current:
    parent: "150 Custom Plugins and Middleware"
---
The `krakend check-plugin` command helps you validate the **compatibility of your custom plugins** that will run in conjunction with KrakenD.

The command compares your plugin's `go.sum` file with the libraries initially used to compile the running binary. If there are any incompatibilities between your plugin and KrakenD, it will show a detailed list.

If you integrate this command as part of your CI/CD pipeline, it will exit with a status code `0` when the libraries of your plugin are compatible with KrakenD and with a status code `1` when they are not.

Notice that the `check-plugin` command does not check the plugin's validity itself nor need its source code other than the `go.sum` file.

To get started writing your plugins see:

- [Introduction to custom plugins](/docs/extending/introduction/)
- [Writing custom plugins](/docs/extending/writing-plugins/)

The `krakend check-plugin` command accepts the following options:

{{< terminal title="Usage of KrakenD check" >}}
./krakend check-plugin -h

{{< ascii-logo >}}

Version: {{< product latest_version >}}

Validates that the active configuration file has a valid syntax to run the service.
Change the configuration file by using the --config flag

Usage:
  krakend check-plugin [flags]

Examples:
krakend check-plugin --go 1.17.7 --libc MUSL-1.2.2 --sum ./go.sum

Flags:
  -h, --help          help for plugin
  -s, --sum string    Path to the go.sum file to analize (default "./go.sum")
  -g, --go string     Version of the go compiler used (default "1.17.7")
  -l, --libc string   Version of the libc library used
{{< /terminal >}}

## Flags
Use `krakend check-plugin` in combination with the following flags:

- `-s` or `--sum` to specify the path to the `go.sum` file of your plugin.
- `-g` or `--go` to specify the Go version you are using to compile the plugin.
- `-l` or `--libc` to specify the libc version installed in the system. The libc version must have the preffix `MUSL-`, `GLIBC-`, `DARWIN-`. For instance, a plugin in Mac Monterrey might use `DARWIN-12.2.1`, an Alpine container will need something like `MUSL-1.2.2`, and a Linux box will have `GLIBC-2.32`.


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
