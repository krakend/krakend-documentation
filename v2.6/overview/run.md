---
lastmod: 2016-10-28
old_version: true
date: 2016-10-28
linktitle: Running KrakenD
title: Running KrakenD server
description: All options of the krakend command to manage the service, write plugins, or get information about the installed version.
weight: 20
notoc: true
menu:
  community_v2.6:
    parent: "000 Getting Started"
---

After installing KrakenD, you can start using KrakenD by typing `krakend`. To see all the options of `krakend`, type `krakend -h` or `krakend <COMMAND> -h`. For instance, the `krakend run` help is:

{{< terminal title="Run command help" >}}
krakend run -h

{{< ascii-logo >}}

Version: 2.6

The API Gateway builder

Usage:
  krakend [command]

Available Commands:
  audit         Checks the integrity of the config and returns security recommendations.
  check         Validates that the configuration file is valid.
  check-plugin  Check the compatibility with the plugin deps.
  help          Help about any command
  run           Run the KrakenD server.
  version       Shows KrakenD version.

Flags:
  -c, --config string   Path to the configuration filename
  -d, --debug           Enable the debug
  -h, --help            help for krakend

Use "krakend [command] --help" for more information about a command.
{{< /terminal >}}

You can use the following commands:

- `krakend audit`: Use [krakend audit](/docs/v2.6/configuration/audit/) to get security recommendations for a given configuration.
- `krakend check`: Use [krakend check](/docs/v2.6/configuration/structure/) to make sure the configuration file you have generated is not broken and has the required attributes to start the gateway.
- `krakend check-plugin`: Use the [check-plugin](/docs/v2.6/extending/check-plugin/) when you are developing custom plugins and you want to check that they are compatible with the server.
- `krakend run`: Use run to start the API gateway server.
- `krakend version`: Use the version command to print the current KrakenD version and the Glibc and Go versions used during compilation.

## Starting the gateway server
To start the server, invoke the `krakend run` command with a configuration file containing your API definition. You can visually create your first `krakend.json` file using the [KrakenDesigner](https://designer.krakend.io/) if you prefer a UI.

Or to get started right away, you can paste the following content inside a `krakend.json` file:

```json
{
    "$schema": "https://www.krakend.io/schema/v2.6/krakend.json",
    "version": 3
}
```

And then you can start KrakenD:

{{< terminal title="Command to start KrakenD" >}}
krakend run -c krakend.json
{{< /terminal >}}

Or if you use Docker:

{{< terminal title="Command to start KrakenD with Docker" >}}
docker run -p "8080:8080" -v $PWD:/etc/krakend/ {{< product image >}}:2.6 run -c krakend.json
{{< /terminal >}}

Now KrakenD is listening on `8080`, and you can see it working under `http://localhost:8080/__health`.
