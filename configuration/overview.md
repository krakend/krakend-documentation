---
lastmod: 2018-09-21
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
When the KrakenD service is started, the `run` command requires passing the flag `-c` indicating the path to the configuration file that defines KrakenD behavior. This configuration file is usually referred through the documentation as the `krakend.json`. This is just a convenient name for the documentation but the configuration file doesn't need to live or be named in any specific way.

If your configuration file is complex it can be split into multiple files with dynamic values where ENV vars can be injected (see [flexible-config](/docs/configuration/flexible-configuration)).

Provided this simple configuration mechanism, the versioning and automation are very convenient. Any change in the API Gateway will be always under version control and it can be reverted to a previous state without effort.

# Generating the configuration file
The configuration file can be written from scratch or reuse another existing file as a base, but the easiest way to write your first configuration file is by simply using the online configuration editor [KrakenDesigner](http://designer.krakend.io/).

The KrakenDesigner is a simple javascript application that helps you understand the capabilities of the API Gateway and helps you set the different values for all the different options. Using this option you don't need to learn and write from scratch all the attribute names. The configuration file can be downloaded at any time and loaded again to resume the edition.

The Kraken Designer is a **pure static** page that **does not send any of your configuration elsewhere**, and as it happens with all our software, is also open sourced and you can download it and run it in your own web server. See the [Krakendesigner](https://github.com/devopsfaith/krakendesigner) repository.

<a class="btn btn-secondary btn-circle" href="http://designer.krakend.io/">Generate configuration now</a>

# Supported file formats
The KrakenD configuration file can be encoded as:

- `yaml`
- `toml`
- `json`

Our recommendation is to **choose JSON**, at least until you are familiarized with the product. If you intend to autogenerate your configuration file using the KrakenDesigner, the output will be `JSON`, and if you want to continue later editing a configuration file again with the designer, it only can read JSON as well.

Worth mentioning that all our examples in the documentation and repositories are today shown only in `JSON` format so it will be always easier for you to copy/paste snippets of code. But in any case, you are free to choose `yaml` or `toml` if you are more comfortable with this options.

# Validating the syntax of the configuration file
Let's say you are modifying by hand the file `~/my-krakend.json`. When you are set, you can check the syntax of the file by running:

    krakend --config ~/my-krakend.json check

By adding the flag `--debug` you will be able to see the full output with the interpretation of the file:

    krakend --config ~/my-krakend.json --debug

Of course you can try to start the service directly as this will be done anyway. When the syntax is correct you'll see the message

    Syntax OK!

The configuration file contains a lot of different options that are not explained in this section, the best way to get familiar with them is using the KrakenDesigner.
