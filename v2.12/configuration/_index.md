---
lastmod: 2025-10-16
old_version: true
date: 2016-07-01
linktitle: Configuration overview
title: Configuration Guide for KrakenD API Gateway
description: Get a comprehensive guide on configuring KrakenD API Gateway to optimize and streamline your API management processes
menu:
  community_v2.12:
    parent: "010 Configuration files"
weight: -1000
dark_header_image: true
images:
- /images/documentation/hero/configuration.png
---
All the setup a KrakenD server needs to operate is a single configuration file. This file is referred to as `krakend.json` throughout all the documentation.

The name `krakend.json` is simply an **alias**, a convention that we use everywhere. Your real configuration file can have any name, multiple formats (such as YAML or TOML), be stored anywhere, or be **split into numerous pieces**.

With this simple configuration mechanism in place, **versioning and automation are very convenient**. Any change to the API Gateway or the AI Gateway is always under version control, and the code controls the state of the gateway.

{{< note title="Configuration using multiple files" type="tip" >}}
If your configuration file is too large or repetitive, or is maintained by many people, it can be split into several files using a templating system. See the [flexible configuration documentation](/docs/v2.12/configuration/flexible-config/) for more information.
{{< /note >}}


## Generating the configuration file(s)
The configuration file is how you provide instructions to KrakenD. To start your first KrakenD configuration, you can:

- Use the [KrakenD Playground](/docs/v2.12/overview/playground/) and start editing its configuration from a working environment with several use-case examples.
- Use the online configuration editor [KrakenDesigner](/docs/v2.12/configuration/designer/).
- Start a file from scratch or from a template

The [KrakenDesigner](/docs/v2.12/configuration/designer/) is a simple javascript application that helps you understand some of the capabilities of the API Gateway and helps you set the different values for all the different options. Using this option, you don't need to learn and write from scratch all the attribute names. You can download the configuration file at any time and load it to resume editing. The Kraken Designer is a **pure static** page that **does not send any of your configuration elsewhere**.

{{< button-group >}}
{{< button url="https://designer.krakend.io/" text="Generate configuration now" >}}
{{< /button >}}
{{< /button-group >}}

To start editing from scratch, use a [modern IDE with JSON Schema integration](/docs/v2.12/developer/ide-integration/), and play with the autocomplete! Here's a starting point:

```json
{
  "$schema": "https://www.krakend.io/schema/v2.12/krakend.json",
  "version": 3,
  "debug_endpoint": true,
  "echo_endpoint": true,
  "endpoints": [
    {
      "endpoint": "/test",
      "backend": [
        {
          "@comment": "Connect KrakenD to KrakenD itself as backend!",
          "host": [
            "http://localhost:8080"
          ],
          "url_pattern": "/__echo/test"
        }
      ]
    }
  ]
}
```

Start KrakenD and connect to `http://localhost:8080/test` or `http://localhost:8080/__health` now.

## Understanding the configuration parts
Whether you choose to start with the KrakenDesigner, a template, or from scratch, we recommend that you understand the configuration structure and edit the file(s) by hand as soon as possible. Always [run the linter](/docs/v2.12/configuration/check/) after you finish editing to make sure you have a valid configuration.

Read the [introduction to the configuration file](/docs/v2.12/configuration/structure/) to understand the different parts of the configuration.

## Supported file formats
Throughout all the documentation, we refer to the configuration file as the `krakend.json` file; however, you can also express the configuration file using `.json`, `.toml`, or `.yaml` extensions. For more information and recommendations, see [supported file formats](/docs/v2.12/configuration/supported-formats/).

## Validating the syntax of the configuration file
Validate the syntax (not the logic) of your configuration file using the `krakend check` command:

{{< terminal title="Check the configuration" >}}
krakend check --lint --config ./krakend.toml --debug
{{< /terminal >}}

When the syntax is correct, you'll see the message `Syntax OK!`; otherwise, the error is displayed.

You can, of course, start the service directly without checking, and KrakenD will do everything possible to start the service (even if there are configuration inconsistencies, at your own risk), but **never send a configuration to production without passing the lint first**

Read more about [`krakend check`](/docs/v2.12/configuration/check/)
