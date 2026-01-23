---
lastmod: 2022-10-10
old_version: true
date: 2020-07-10
linktitle: Environment vars
menu:
  community_v2.12:
    parent: "010 Configuration files"
title: Environment variables in the configuration
description: Learn how to leverage environment variables in KrakenD API Gateway configuration for flexible and dynamic configuration management
weight: 40
meta:
  since: v1.2
  source: false
  namespace: false
  scope:
  - service
---
When KrakenD runs (whether with `run` or `check`), all the behavior is loaded from the [configuration file](/docs/v2.12/configuration/structure/). Through environment variables, you can also set values. There are two different ways of injecting environment vars:

- **Use a `KRAKEND_`-like reserved environment variable**: To override values set in the configuration.
- **Set your own environment variables** when using the `{{env}}` function in [flexible configuration](/docs/v2.12/configuration/flexible-config/) templates.

## Use a reserved environment variable
There are a group of reserved environment variables that are automatically recognized by KrakenD when set.

Examples are when you want to replace the `port`, the default `timeout`, or the configuration `name` (sent to your telemetry) that already exists in the configuration.

In essence, you can replace any value in the configuration that lives in the root level, and is a string, an integer, or a boolean. To do it you only need to capitalize the property name and add a prefix `KRAKEND_`.

{{< note title="Reserved variables are ignored unless the key exists in the configuration" type="warning" >}}
The following list of variables only set the desired values when you have its associated value in the configuration. They are meant to **override** settings **already present** in the configuration, but if you set one of them and there is no value in the configuration, it won't have any effect.
{{< /note >}}


{{< top_level_envvars >}}

### Reserved variable example
For instance, take the following `krakend.json` configuration as an example:

```json
{
    "version": 3,
    "timeout": "3s",
    "name": "Example gateway.",
    "cache_ttl": "0s"
}
```

You could start the server with the following command which would allow you to override the values in the configuration:

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
**Important**: Notice that the `port` attribute is not present in the configuration, despite passing a `KRAKEND_PORT` parameter. This is because the `port` didn't exist previously in the configuration file, and the environment variables can only **override** values.


## Setting your environment variables
If you need to set content using environment variables at any level, you have can either use the [flexible configuration](/docs/v2.12/configuration/flexible-config/), which includes a series of [advanced functions](/docs/v2.12/configuration/templates/#sprig-functions) including an `env` function, or you can not use KrakenD at all and rely on the operating system `envsubst` command. Obviously you can also write your custom replacement process.

### Environment variables with Flexible Configuration
Here is an example with Flexible Configuration:

```go-text-template
{
    "version": 3,
    "name": "Configuration for {{ env "MY_POD_NAMESPACE" }}"
}
```
When you use the flexible configuration, you can start KrakenD from the template that uses them.

### Environment variables with envsubst
Another example is not to use any of the built-in features of KrakenD and rely on your operating system via the command `envusbst`.

For instance, you have a configuration file `krakend.template.json` like the following:

```json
{
    "version": 3,
    "name": "Configuration for $MY_POD_NAMESPACE"
}
```
Then you can generate the final configuration `krakend.json` like this:

{{< terminal title="Environment variable substitution" >}}
export MY_POD_NAMESPACE="my-namespace" && envsubst < krakend.template.json > krakend.json
{{< /terminal >}}

The command, which is generally available in Linux distributions, takes a template file as input and outputs the same file with the environment variables replaced (you cannot override the same file). You have to be aware that missing variables are simply replaced by an empty string.

Note: on Alpine-based containers, like the KrakenD image, you need to do an `apk add envsubst` to use this command.