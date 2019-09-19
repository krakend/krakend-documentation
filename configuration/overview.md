---
lastmod: 2018-10-20
date: 2016-07-01
linktitle: Configuration overview
menu:
  documentation:
    parent: configuration
title: KrakenD's configuration file(s)
weight: -1000
aliases:
- /docs/overview/configuration/
---
All the configuration that the KrakenD server needs to start and operate is a single configuration file. This file is referred to as `krakend.json` through all the documentation.

The name `krakend.json` is just **an alias**, a convention, that we use everywhere. Your real configuration file can have any name, be stored anywhere, or split in many pieces.

Provided this simple configuration mechanism, the **versioning and automation are very convenient**. Any change in the API Gateway is always under the version control system, and the code controls the state of the gateway.


{{% note title="Configuration using multiple files" %}}
If your configuration file is too large or repetitive, it can be split into several files using a templating system. See the [flexible configuration documentation](/docs/configuration/flexible-config) for more information on this feature.
{{% /note %}}


# Generating the configuration file(s)
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
