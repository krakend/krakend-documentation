---
aliases: ["/overview/usage/"]
lastmod: 2018-09-27
date: 2016-10-25
menu:
  community_current:
    parent: "000 Getting Started"
next: /overview/configuration
notoc: true
prev: /overview/installing
title: Using KrakenD
weight: 30
---

From an operations point of view KrakenD, is very simple to use. It only requires you to pass
the path the configuration file (which defines behaviors and endpoints). Additionally, you can
enable the debug with the `-d` flag, and that's pretty much everything.

## TL;DR
1. Generate a configuration file with your endpoints definition. The easier way to generate it is using the [designer](https://designer.krakend.io/)
2. Check the syntax of your `krakend.json` is good
	{{< terminal title="Syntax checking" >}}
krakend check --config krakend.json --debug
	{{< /terminal >}}
3. Run KrakenD
	{{< terminal title="Start the server" >}}
krakend run -c krakend.json -d
	{{< /terminal >}}


The flag `-c` is the short version of `--config` and `-d` the short version of `--debug` which allows
you to find problems easily.


## Using KrakenD

To start KrakenD make sure to provide the path to the binary or add it to the `PATH`. You can see all
KrakenD options by running the binary with no options:

{{< terminal title="The krakend command" >}}
krakend

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
  check       Validates that the configuration file is valid.
  help        Help about any command
  run         Run the KrakenD server.

Flags:
  -c, --config string   Path to the configuration filename
  -d, --debug           Enable the debug
  -h, --help            help for krakend

Use "krakend [command] --help" for more information about a command.
{{< /terminal >}}

As you can see there are 3 different supported commands:

- [`krakend check`](/docs/commands/check/) (syntax validation)
- [`krakend run`](/docs/commands/run/) (run the server)
- `krakend help` (show usage)
