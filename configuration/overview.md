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
The KrakenD configuration file can be encoded as:

- `yaml`
- `toml`
- `json`
- `hcl`
- `java.properties`

Our recommendation is to **choose JSON**, at least until you are familiar with the product. If you intend to autogenerate your configuration file using the KrakenDesigner, the output is a `JSON`, and if you want to continue later editing a configuration file again with the designer, it only can read JSON as well.

Worth mentioning that all our examples in the documentation and repositories are today shown in `JSON` format, so it's always easier for you to copy/paste snippets of code. However, you are free to choose `yaml` or `toml` if you are more comfortable with these options.

# Validating the syntax of the configuration file
Let's say you are modifying by hand the file `~/my-krakend.json`. When you finish, you can check the syntax of the file by running:

    krakend --config ~/my-krakend.json check

By adding the flag `--debug` you can see the full output with the interpretation of the file:

    krakend --config ~/my-krakend.json --debug

You can also start the service directly as this is done right before the server starts. When the syntax is correct, you'll see the message

    Syntax OK!

The configuration file contains a lot of different options that are not explained in this section, the best way to get familiar with them is using the KrakenDesigner.
