---
lastmod: 2023-01-31
date: 2016-07-01
linktitle: Supported file formats
title: KrakenD file supported formats
description: What configuration file format should I choose? KrakenD supports many formats but the json format will work best.
menu:
  community_current:
    parent: "010 Configuration file(s)"
notoc: true
weight: 30
---

The expected configuration file format by default is `json`, but KrakenD can parse any of these file formats:

- `.json` (recommended)
- `.toml`
- `.yaml`
- `.yml`
- `.properties`
- `.props`
- `.prop`
- `.hcl`

You can validate the syntax of any of these with [`krakend check`](/docs/configuration/check/)

## Why is JSON recommended?
You are free to choose `json`, `yaml`, `toml` or any of the other formats listed above at your best convenience. But have in mind the following limitations when choosing a file format that is not `json`:

- **Using the UI**: If you intend to generate or edit your configuration file using the [KrakenDesigner](https://designer.krakend.io), the input and output are always a `.json` file.
- **Flexible Configuration**: If you want to split the configuration file into different pieces or use variables inside the configuration, the [flexible configuration](/docs/configuration/flexible-config/) works better with `json`, as formats that need a proper indentation present more problems to control it (e.g.: `yaml`).
- **Documentation**: All our examples in the documentation and repositories are today shown in `json` format, so it's always more convenient to reuse snippets of code (copy and paste!).
- **Linting**: You cannot use the `--lint` flag on the [check command](/docs/configuration/check/) because expects json content to validate its schema (you could convert the file using a 3rd party tool though). You can still use the rest of the check commands.

As you can see, if you are OK with the limitations above, you can freely choose any other format.
