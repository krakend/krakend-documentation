---
lastmod: 2020-07-10
date: 2020-07-10
linktitle: Environment vars
since: 1.2
notoc: true
menu:
  documentation:
    parent: configuration
title: Overriding the configuration with environment vars
weight: 40
---
When KrakenD [runs](/docs/commands/run), all the behavior is loaded from the [configuration file](/docs/configuration/structure). For each configuration value that isn't nested (meaning first-level properties of the configuration), you can override its value with an environment variable.

All configuration environment variables must have the prefix `KRAKEND_` and declared in uppercase. The variable name after the prefix must match the property in the configuration value.

For instance, take the following `krakend.json` configuration as an example:

    {
        "version": 2,
        "timeout": "3s",
        "name": "Example gateway."
    }

Now run krakend with the following command:

{{< terminal title="Example: Override configuration with env vars" >}}
KRAKEND_NAME="Build ABC0123" KRAKEND_TIMEOUT="500ms" krakend run -c krakend.json
{{< /terminal >}}

The resulting configuration will be:

    {
        "version": 2,
        "timeout": "500ms",
        "name": "Build ABC0123"
    }

**NOTE**: The configuration file is not changed. The values above are a representation of the final mapped values.