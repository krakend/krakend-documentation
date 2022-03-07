---
lastmod: 2018-10-28
old_version: true
date: 2016-07-01
linktitle: Supported file formats
menu:
  community_v1.4:
    parent: "010 Configuration file(s)"
title: KrakenD file supported formats
notoc: true
weight: 30
---

The expected configuration file format by default is `json`, but KrakenD can parse different formats if one of the following is found:

- `krakend.json`
- `krakend.toml`
- `krakend.yaml`
- `krakend.yml`
- `krakend.properties`
- `krakend.props`
- `krakend.prop`
- `krakend.hcl`

Nevertheless, **our recommendation is to choose `JSON`**.

Validate the syntax (not the logic) with [`krakend check`](/docs/v1.4/commands/check/)

## Why choosing json?
You are free to choose `YAML`, `TOML` or any of the other formats at your best convenience. But have in mind the following logic when choosing a file format other than `json`.

**Using the UI**: If you intend to generate or edit your configuration file using the [KrakenDesigner](https://designer.krakend.io), the input and output are always a `.json` file.

**Flexible Configuration**: If you want to split the configuration file into different pieces or use variables inside the configuration, the [flexible configuration](/docs/v1.4/configuration/flexible-config/) needs `JSON`.

**Documentation**: All our examples in the documentation and repositories are today shown in `JSON` format, so it's always more convenient to reuse snippets of code.
