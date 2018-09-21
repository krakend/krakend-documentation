---
aliases:
- /commands/run
lastmod: 2016-10-28
date: 2016-10-28
linktitle: Run
title: Running KrakenD server. The `krakend run` command
description: KrakenD Run Command
weight: 1
notoc: true
menu:
  documentation:
    parent: Command line commands
---

To start KrakenD you need to invoke the `run` command with the path to the configuration file. You
can also specify the port (defaults to `8080`)

    krakend run -c krakend.json
    # or
    krakend run --config /path/to/krakend.json
    # or
    krakend run --config /path/to/krakend.json -p 8080

The `krakend run` command with no flags will remind you that you need the path to the configuration file:

    $ ./krakend run
    Please, provide the path to your config file

Show the help:

    $  ./krakend run -h

    `7MMF' `YMM'                  `7MM                         `7MM"""Yb.
      MM   .M'                      MM                           MM    `Yb.
      MM .d"     `7Mb,od8 ,6"Yb.    MM  ,MP'.gP"Ya `7MMpMMMb.    MM     `Mb
      MMMMM.       MM' "'8)   MM    MM ;Y  ,M'   Yb  MM    MM    MM      MM
      MM  VMA      MM     ,pm9MM    MM;Mm  8M""""""  MM    MM    MM     ,MP
      MM   `MM.    MM    8M   MM    MM `Mb.YM.    ,  MM    MM    MM    ,dP'
    .JMML.   MMb..JMML.  `Moo9^Yo..JMML. YA.`Mbmmd'.JMML  JMML..JMMmmmdP'
    _______________________________________________________________________

    Version: 0.4

    Run the KrakenD server.

    Usage:
      krakend run [flags]

    Examples:
    krakend run -d -c config.json

    Flags:
      -h, --help       help for run
      -p, --port int   Listening port for the http service

    Global Flags:
      -c, --config string   Path to the configuration filename
      -d, --debug           Enable the debug



## Example
The most common way of starting the service is:

    krakend run --config krakend.json

To start the KrakenD service in a different port:

    krakend run --config path/to/krakend.json --port 8888

In development and testing phase increase the verbosity to see what is going on with:

    krakend run --log INFO