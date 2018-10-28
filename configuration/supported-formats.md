---
lastmod: 2018-10-20
date: 2016-07-01
linktitle: Supported file formats
menu:
  documentation:
    parent: configuration
title: KrakenD file supported formats
notoc: true
weight: 20
---

The expected configuration file format by default is `json`, but KrakenD can parse different formats if one of the following extensions is found:

- `.json`
- `.toml`
- `.yaml`
- `.yml`
- `.properties`
- `.props`
- `.prop`
- `.hcl`

Nevertheless, **our recommendation is to choose `JSON`**. 

Validate the syntax (not the logic) with [`krakend check`](/docs/commands/check/)

# Why choosing json?
You are free to choose `YAML`, `TOML` or any of the other formats at your best convenience. But have in mind the following logic when choosing a file format other than `json`. 

**Using the UI**: If you intend to generate or edit your configuration file using the [KrakenDesigner](https://designer.krakend.io), the input and output are always a `.json` file.

**Documentation**: All our examples in the documentation and repositories are today shown in `JSON` format, so it's always more convenient to reuse snippets of code.
