---
lastmod: 2016-10-28
old_version: true
date: 2016-10-28
linktitle: Run
title: Running KrakenD server. The `krakend run` command
description: KrakenD Run Command
weight: 1
notoc: true
menu:
  community_v1.3:
    parent: "020 Command Line"
---

To start KrakenD, you need to invoke the `run` command with the path to the configuration file. You
can also specify the port (defaults to `8080`)

{{< terminal title="Command to start KrakenD" >}}
krakend run -c krakend.json
# or
krakend run --config /path/to/krakend.json
# or
krakend run --config /path/to/krakend.json -p 8080
{{< /terminal >}}

The `krakend run` command with no flags will remind you that you need the path to the configuration file:

{{< terminal title="Missing configuration file" >}}
krakend run
Please, provide the path to your config file
{{< /terminal >}}

Show the help:
{{< terminal title="Run command help" >}}
krakend run -h

`7MMF' `YMM'                  `7MM                         `7MM"""Yb.
  MM   .M'                      MM                           MM    `Yb.
  MM .d"     `7Mb,od8 ,6"Yb.    MM  ,MP'.gP"Ya `7MMpMMMb.    MM     `Mb
  MMMMM.       MM' "'8)   MM    MM ;Y  ,M'   Yb  MM    MM    MM      MM
  MM  VMA      MM     ,pm9MM    MM;Mm  8M""""""  MM    MM    MM     ,MP
  MM   `MM.    MM    8M   MM    MM `Mb.YM.    ,  MM    MM    MM    ,dP'
.JMML.   MMb..JMML.  `Moo9^Yo..JMML. YA.`Mbmmd'.JMML  JMML..JMMmmmdP'
_______________________________________________________________________

Version: 1.3

The API Gateway builder

Usage:
  krakend [command]

Available Commands:
  check       Validates that the configuration file is valid.
  help        Help about any command
  run         Run the KrakenD server.

Flags:
  -c, --config string   Path to the configuration filename
  -d, --debug           Enable the debug
  -h, --help            help for krakend

Use "krakend [command] --help" for more information about a command.
{{< /terminal >}}



## Example
The most common way of starting the service is:

{{< terminal title="Start krakend" >}}
krakend run --config krakend.json
{{< /terminal >}}

To start the KrakenD service in a different port (the port can be set in the configuration file as well):

{{< terminal title="Start in a custom port" >}}
krakend run --config path/to/krakend.json --port 8888
{{< /terminal >}}

In development and testing phase [increase the verbosity of the logs](/docs/v1.3/logging/extended-logging/#set-the-reporting-level)
