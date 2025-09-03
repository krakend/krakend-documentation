---
lastmod: 2022-01-19
old_version: true
date: 2020-07-10
linktitle: Environment vars
since: 1.2
notoc: true
menu:
  community_v2.0:
    parent: "010 Configuration file(s)"
title: Overriding the configuration with environment vars
weight: 40
meta:
  since: v1.2
  source: false
  namespace: false
  scope:
  - service
---
When KrakenD runs, all the behavior is loaded from the [configuration file](/docs/v2.0/configuration/structure/). Through environment variables, you can override existing values in the configuration. There are two different ways of injecting environment vars.

{{< note title="The key to override must exist in the configuration" type="note" >}}
The environment variables are meant to **replace** existing attributes in the configuration. Therefore, you cannot set new parameters that do not exist in the configuration.
{{< /note >}}


## First level properties
You can override its value with an environment variable for each configuration value that isn't nested (meaning **first-level properties of the configuration**).

All configuration parameters you want to set using environment variables, pass them with a prefix `KRAKEND_`. The variable name after the prefix must match the property in the configuration value using uppercase.

For instance, take the following `krakend.json` configuration as an example:

{{< highlight json >}}
{
    "version": 3,
    "timeout": "3s",
    "name": "Example gateway."
}
{{< /highlight >}}


To replace values using env vars, run krakend with the following command:

{{< terminal title="Example: Override configuration with env vars" >}}
KRAKEND_NAME="Build ABC0123" KRAKEND_TIMEOUT="500ms" KRAKEND_PORT=9000 krakend run -c krakend.json
{{< /terminal >}}

The resulting configuration will be:

{{< highlight json >}}
{
    "version": 3,
    "timeout": "500ms",
    "name": "Build ABC0123"
}
{{< /highlight >}}

Notice that the `port` attribute is not present in the configuration, despite passing a `KRAKEND_PORT` parameter. This is because the `port` didn't exist previously in the configuration file, and the environment variables can only replace values.


**NOTE**: The configuration file is not changed. The values above are a representation of the final mapped values.

## Overriding properties in any nesting level
If you need to replace content using environment variables at any level, you have to use the [flexible configuration](/docs/v2.0/configuration/flexible-config/). It includes a series of [advanced functions](/docs/v2.0/configuration/flexible-config/#advanced-functions) including an `env` function that can write in the config any value.
{{< highlight go-text-template >}}
{
    "version": 3,
    "name": "Configuration for {{ env "MY_POD_NAMESPACE" }}"
}
{{< /highlight >}}

## Usage reporting env var
When KrakenD starts, sends a request to our stats server with anonymous non-sensitive information. Our Telemetry system sends **1 request every 12 hours** and contains the following data:

- The KrakenD Version you are running
- The architecture (e.g: `amd64`)
- The operating system /`linux`/ `darwin`)
- A random unique ID

That's all we collect. We are well aware of the importance of privacy. We are not in the data-mining business, so we selected a set of minimal details to share from your KrakenD instances that would give us enough insights into the matter without being invasive. We decided that we'd rather lose some accuracy than collect (maybe) sensible information, so we went for this **anonymous approach**.

We don't collect typical system metrics like the number of CPU/cores, CPU usage, available and consumed ram, network throughput, etc. That’s something more related to system monitoring than to the use of KrakenD, and we felt that collecting these metrics generates friction with the acceptance of a telemetry system.

In any case if you are not comfortable with sharing your KrakenD version, in the open source version you can disable it by passing an environment variable `USAGE_DISABLE=1`.

If you are curious to know how is built, [read this blog post](/blog/building-a-telemetry-service/).