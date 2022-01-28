---
aliases: ["/commands/run"]
lastmod: 2016-10-28
date: 2016-10-28
linktitle: Running KrakenD
title: Running KrakenD server. The `krakend run` command
description: KrakenD Run Command
weight: 1
notoc: true
menu:
  community_current:
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

Version: {{< version >}}

The API Gateway builder

Usage:
  krakend [command]

Available Commands:
  check         Validates that the configuration file is valid.
  check-plugin  Check the compatibility with the plugin deps.
  help          Help about any command
  run           Run the KrakenD server.

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

In development and testing phase [increase the verbosity of the logs](/docs/logging/extended-logging/#set-the-reporting-level)

## Anonymous reporting
When KrakenD starts, sends a request to our stats server with anonymous no-sensitive information. Our Telemetry system sends **1 request every 12 hours** and contains the following data:

- The KrakenD Version you are running
- The architecture (e.g: `amd64`)
- The operating system /`linux`/ `darwin`)
- A random unique ID

That's all we collect. We are well aware of the importance of privacy. We are not in the data-mining business, so we selected a set of minimal details to share from your KrakenD instances that would give us enough insights into the matter without being invasive. We decided that we'd rather lose some accuracy than collect (maybe) sensible information, so we went for this **anonymous approach**.

We don't collect typical system metrics like the number of CPU/cores, CPU usage, available and consumed ram, network throughput, etc. That’s something more related to system monitoring than to the use of KrakenD, and we felt that collecting these metrics generates friction with the acceptance of a telemetry system.

In any case if you are not comfortable with sharing your KrakenD version, in the open source version you can disable it by passing an environment variable `USAGE_DISABLE=1`.

If you are curious to know how be built it, [read our blog post](/blog/building-a-telemetry-service/).