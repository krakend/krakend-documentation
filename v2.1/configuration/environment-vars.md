---
lastmod: 2022-10-10
old_version: true
date: 2020-07-10
linktitle: Environment vars
menu:
  community_v2.1:
    parent: "010 Configuration file(s)"
title: Overriding configuration with environment vars
weight: 40
meta:
  since: 1.2
  source: false
  namespace: false
  scope:
  - service
---
When KrakenD runs, all the behavior is loaded from the [configuration file](/docs/v2.1/configuration/structure/). Through environment variables, you can override some of its values. There are two different ways of injecting environment vars.

- **Replacing existing values** in the configuration
- **Setting new values** when using the `{{env}` function in [flexible configuration](/docs/v2.1/configuration/flexible-config/)

## Value replacement with env vars
You can **override** configuration values with an environment variable for each configuration value that isn't nested (meaning **first-level properties** of the configuration). However, with this technique, you cannot override parameters that aren't declared in the configuration.

Examples of values you can replace are the `port`, `timeout`, or the configuration `name` to name a few.

To replace configuration parameters during runtime, capitalize them and add a prefix `KRAKEND_`. The configuration file is not changed. Only the values hold in memory.

For instance, take the following `krakend.json` configuration as an example:

```json
{
    "version": 3,
    "timeout": "3s",
    "name": "Example gateway.",
    "cache_ttl": "0s"
}
```

To replace values using env vars, you could start krakend with the following command:

{{< terminal title="Example: Override configuration with env vars" >}}
KRAKEND_NAME="Build ABC0123" \
KRAKEND_TIMEOUT="500ms" \
KRAKEND_PORT=9000 \
krakend run -c krakend.json
{{< /terminal >}}

The resulting configuration will be:

```json
{
    "version": 3,
    "timeout": "500ms",
    "name": "Build ABC0123"
}
```


Notice that the `port` attribute is not present in the configuration, despite passing a `KRAKEND_PORT` parameter. This is because the `port` didn't exist previously in the configuration file, and the environment variables can only override values.

## Setting new values
If you need to set content using environment variables at any level, you have to use the [flexible configuration](/docs/v2.1/configuration/flexible-config/). It includes a series of [advanced functions](/docs/v2.1/configuration/flexible-config/#advanced-functions) including an `env` function that can write in the config any value.

Here is an example:

```go-text-template
{
    "version": 3,
    "name": "Configuration for {{ env "MY_POD_NAMESPACE" }}"
}
```

## Usage reporting env var
When KrakenD starts, it sends a request to our stats server with anonymous non-sensitive information. Our Telemetry system sends **1 request every 12 hours** and contains the following data:

- The KrakenD Version you are running
- The architecture (e.g: `amd64`)
- The operating system /`linux`/ `darwin`)
- A random unique ID

That's all we collect ([source code here](https://github.com/krakend/krakend-usage)). We are well aware of the importance of privacy. However, we are not in the data-mining business, so we selected a set of minimal details to share from your KrakenD instances that would give us enough insights into the matter without being invasive. We decided that we'd rather lose some accuracy than collect (maybe) sensible information, so we went for this **anonymous approach**.

We don't collect typical system metrics like the number of CPU/cores, CPU usage, available and consumed ram, network throughput, etc. That’s something more related to system monitoring than KrakenD, and we felt that collecting these metrics generates friction with the acceptance of a telemetry system.

If you are not comfortable sharing your KrakenD version, you can disable it in the open-source version by passing an environment variable `USAGE_DISABLE=1`. If you want to know how it's built, [read this blog post](/blog/building-a-telemetry-service/).