---
lastmod: 2018-10-20
date: 2016-07-01
linktitle: Configuration overview
menu:
  documentation:
    parent: configuration
title: KrakenD's configuration file overview
weight: -1000
aliases:
- /docs/overview/configuration/
---
When the KrakenD service is about to be started, the `run` command requires passing the flag `-c` which states the path to the configuration file. We refer through the documentation to this file as the `krakend.json`, but it can be stored or named according to your preferences.

If your configuration file is too large or repetitive, it can be split into multiple files using a templating system, where ENV vars can also be injected as dynamic values. Although this is not part of KrakenD-CE itself, you might be interested in using the [flexible-config](/docs/configuration/flexible-configuration)).

Provided this simple configuration mechanism, the versioning and automation are very convenient. Any change in the API Gateway is always under the version control system, and the code controls the state of the gateway.

# Generating the configuration file
The configuration file can be written from scratch or reuse another existing file as a base, but the easiest way to write your first configuration file is by simply using the online configuration editor [KrakenDesigner](https://designer.krakend.io/).

The KrakenDesigner is a simple javascript application that helps you understand the capabilities of the API Gateway and helps you set the different values for all the different options. Using this option you don't need to learn and write from scratch all the attribute names. The configuration file can be downloaded at any time and loaded again to resume the edition.

The Kraken Designer is a **pure static** page that **does not send any of your configuration elsewhere**, and as it happens with all our software, is also open sourced and you can download it and run it in your own web server. See the [Krakendesigner](https://github.com/devopsfaith/krakendesigner) repository.

<a class="btn btn-secondary btn-circle" href="https://designer.krakend.io/">Generate configuration now</a>

# Supported file formats
The configuration file can be written as `.json`, `.toml`, `.yaml`, `.yml`, `.properties`, `.props`, `.prop` or `.hcl`. For more information and recommendations see [supported file formats](/docs/configuration/supported-formats/).

# Validating the syntax of the configuration file
Validate the syntax (not the logic) of your configuration file using the `krakend check` command:

    krakend check --config ./krakend.toml --debug

You can also start the service directly as this is done right before the server starts. When the syntax is correct, you'll see the message

    Syntax OK!

Read more about [`krakend check`](/docs/commands/check/)