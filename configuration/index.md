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
When the KrakenD service is started, the `run` command requires the path to a configuration file that will define all KrakenD behavior. This configuration file is usually referred as the `/etc/krakend/krakend.json` or just `krakend.json`. This is a convenient name for this documentation but the configuration file can have any path or name you want, and even it can be split into multiple configuration files with dynamic values where ENV vars can be injected (see [flexible-config](/docs/configuration/flexible-configuration)).


# A stateless service
As KrakenD is a stateless service, it doesn't require coordination or knowledge with the other nodes (**no database!**) and every KrakenD node has everything it needs to be configured in the configuration file.

Provided this simple configuration mechanism, the versioning and automation is very convenient. Any change in the API Gateway will be always under version control and it can be reverted to a previous state without effort.

# Generating a configuration file
The configuration file can be written from scratch or reuse another existing file as a base, but the easiest way to write your first configuration file is by simply using the online [KrakenDesigner](http://designer.krakend.io/).

This is a simple javascript application that helps you understand the capabilities of the API Gateway and guides you to set the different values for all the options without learning from scratch the value names for
every attribute. The configuration file can be downloaded at any time and loaded again to resume the edition.

The Kraken Designer is a **pure static** page that **does not send any of your configuration elsewhere**. Of course the designer, as it happens with all our software, is open source and you can download it and run it in your own web server. See the [Krakendesigner](https://github.com/devopsfaith/krakendesigner) repository..

<a class="btn btn-primary btn-circle" href="http://designer.krakend.io/">Generate configuration now</a>

# Supported file formats
The KrakenD framework can read configuration files encoded as:

- `yaml`
- `toml`
- `json`

Our recommendation is to **choose JSON**, at least until you are familiarized with the product. If you intend to autogenerate your configuration file using the KrakenDesigner, the output will be `JSON`, and if you want to continue later editing a configuration file again with the designer, it only can read JSON as well.

In addition to that, all our examples in the documentation and repositories are usually shown in `JSON` format so it will be always for you to copy/paste snippets of code.

# Validating the syntax of the configuration file
Let's say you are modifying by hand the file `~/my-krakend.json`. When you are set, you can check the syntax of the file by running:

    krakend --config ~/my-krakend.json check

By adding the flag `--debug` you will be able to see the full output with the interpretation of the file:

    krakend --config ~/my-krakend.json --debug

Of course you can try to start the service directly as this will be done anyway. When the syntax is correct you'll see the message

    Syntax OK!

The configuration file contains a lot of different options that are not explained in this section, the best way to get familiar with them is using the KrakenDesigner.
