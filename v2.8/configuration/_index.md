---
lastmod: 2018-10-20
old_version: true
date: 2016-07-01
linktitle: Configuration overview
title: Configuration Guide for KrakenD API Gateway
description: Get a comprehensive guide on configuring KrakenD API Gateway to optimize and streamline your API management processes
menu:
  community_v2.8:
    parent: "010 Configuration files"
weight: -1000
---
All the setup a KrakenD server needs to operate is a single configuration file. This file is referred to as `krakend.json` through all the documentation.

The name `krakend.json` is just **an alias**, a convention, that we use everywhere. Your real configuration file can have any name, multiple formats (like YAML or TOML), be stored anywhere, **or split in many pieces**.

Provided this simple configuration mechanism, the **versioning and automation are very convenient**. Any change in the API Gateway is always under the version control system, and the code controls the state of the gateway.


{{< note title="Configuration using multiple files" type="tip" >}}
If your configuration file is too large or repetitive, it can be split into several files using a templating system. See the [flexible configuration documentation](/docs/v2.8/configuration/flexible-config/) for more information on this feature.
{{< /note >}}


## Generating the configuration file(s)
The configuration file can be written from scratch or reuse another existing file as a base, but the easiest way to write your first configuration file is by simply using the online configuration editor [KrakenDesigner](/docs/v2.8/configuration/designer/).

The KrakenDesigner is a simple javascript application that helps you understand the capabilities of the API Gateway and helps you set the different values for all the different options. Using this option you don't need to learn and write from scratch all the attribute names. The configuration file can be downloaded at any time and loaded again to resume the edition.

The Kraken Designer is a **pure static** page that **does not send any of your configuration elsewhere**.

{{< button-group >}}
{{< button url="https://designer.krakend.io/" text="Generate configuration now" >}}
{{< /button >}}
{{< /button-group >}}

## Supported file formats
Through all the documentation we refer to the configuration file as the `krakend.json` file, but the configuration file can be written using `.json`, `.toml`, `.yaml`, `.yml`, `.properties`, `.props`, `.prop` or `.hcl`. For more information and recommendations see [supported file formats](/docs/v2.8/configuration/supported-formats/).

## Validating the syntax of the configuration file
Validate the syntax (not the logic) of your configuration file using the `krakend check` command:

{{< terminal title="Check the configuration" >}}
krakend check --config ./krakend.toml --debug --lint
{{< /terminal >}}

When the syntax is correct, you'll see the message `Syntax OK!`, otherwise the error is shown.

You can also start the service directly as this is done right before the server starts (except the linting).

Read more about [`krakend check`](/docs/v2.8/configuration/check/)
