---
lastmod: 2020-07-10
date: 2020-07-10
linktitle: Environment vars
since: 1.2
notoc: true
menu:
  community_current:
    parent: "010 Configuration file(s)"
title: Overriding the configuration with environment vars
weight: 40
meta:
  since: 1.2
  source: false
  namespace: false
  scope:
  - service
---
When KrakenD [runs](/docs/commands/run/), all the behavior is loaded from the [configuration file](/docs/configuration/structure/). Through environment variables you can inject a value in the configuration when the server starts. There are two different ways of injecting environment vars.

## First level properties
For each configuration value that isn't nested (meaning **first-level properties of the configuration**), you can override its value with an environment variable.

All configuration environment variables that you want to set using environment variables, pass them with a prefix `KRAKEND_`. The variable name after the prefix must match the property in the configuration value using uppercase.

For instance, take the following `krakend.json` configuration as an example:

    {
        "version": 2,
        "timeout": "3s",
        "name": "Example gateway."
    }

To replace values using env vars, run krakend with the following command:

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

## Overriding properties in any nesting level
If you need to replace content using environment variables at any level, you have to use the [flexible configuration](/docs/configuration/flexible-config/). It includes a series of [advanced functions](/docs/configuration/flexible-config/#advanced-functions) including an `env` function that can write in the config any value.

{
    "version": 2,
    "name": "Configuration for {{ env "MY_POD_NAMESPACE" }}"
}
