---
lastmod: 2022-01-28
date: 2022-01-28
linktitle: Checking custom plugins
title: Validate your plugin compatibility
description: KrakenD check-plugin Command
weight: 20
notoc: true
meta:
    since: 2.0
menu:
  community_current:
    parent: "020 Command Line"
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

`7MMF' `YMM'                  `7MM                         `7MM"""Yb.
  MM   .M'                      MM                           MM    `Yb.
  MM .d"     `7Mb,od8 ,6"Yb.    MM  ,MP'.gP"Ya `7MMpMMMb.    MM     `Mb
  MMMMM.       MM' "'8)   MM    MM ;Y  ,M'   Yb  MM    MM    MM      MM
  MM  VMA      MM     ,pm9MM    MM;Mm  8M""""""  MM    MM    MM     ,MP
  MM   `MM.    MM    8M   MM    MM `Mb.YM.    ,  MM    MM    MM    ,dP'
.JMML.   MMb..JMML.  `Moo9^Yo..JMML. YA.`Mbmmd'.JMML  JMML..JMMmmmdP'
_______________________________________________________________________

Version: {{< version >}}

Validates that the active configuration file has a valid syntax to run the service.
Change the configuration file by using the --config flag

Usage:
  krakend check-plugin [flags]

Examples:
krakend check-plugin -v 1.17.0 -s ./go.sum

Flags:
  -h, --help             help for plugin
  -s, --sum string       Path to the go.sum file to analize (default "./go.sum")
  -v, --version string   Version of the go compiler used (default "1.16.4")
{{< /terminal >}}

## Flags
Use `krakend check-plugin` in combination with the following flags:

- `-s` or `--sum` to specify the path to the `go.sum` file of your plugin.
- `-v` or `--version` to specify the Go version you are using to compile the plugin.


{{< terminal title="Checking a failing plugin example" >}}
krakend check-plugin -v 1.17.0 -s ../plugin-tools/go.sum
14 incompatibility(ies) found...
go
    have: 1.17.0
    want: 1.16.4
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
