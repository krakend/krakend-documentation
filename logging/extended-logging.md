---
lastmod: 2019-01-30
date: 2018-10-30
toc: true
linktitle: Logging overview
title: Logging - Syslog, stdout and GELF
weight: 10
source: https://github.com/devopsfaith/krakend-gologging
aliases: ["/docs/logging-metrics-tracing/logging/"]
menu:
  community_current:
    parent: "090 Logging"
---
By default,  when KrakenD starts all the log events are sent to the **standard output** using the basic logger capabilities of the [Lura Project](https://luraproject.org). The reporting level, in that case, is `DEBUG` and adds no prefix to the log lines.

## Extending the logging capabilities

Different logging components allow you to extend the logging functionality, such as sending the events to the **syslog**, choosing the verbosity level, or use the **Graylog Extended Log Format (GELF)**.

In addition to this, a lot of exporters are available to send your logs out (see [Telemetry](/docs/telemetry/overview/))

### Improved logging with `gologging`.

The component `gologging` extends the default logging capabilities with the following capabilities:

- Option to write to the stdout
- Option to write to the syslog
- Add a prefix to log lines
- Select the reporting level
- Option to use a predefined or custom format

### Enabling `gologging`

To enjoy the extended logging capabilities the component needs to be added in the `krakend.json` configuration. Add its namespace in the `extra_config` at the root level:

    {
      "version": 2,
      "extra_config": {
        "telemetry/logging": {
          "level": "INFO",
          "prefix": "[KRAKEND]",
          "syslog": true,
          "stdout": true,
          "format": "custom",
          "custom_format": "%{message}"
        }
      }

The snippet above shows the four options you can configure, explained below.

#### Set the *reporting level*
Define the severity you would like to see in the logs with `level`. The recognized options from more to less verbosity are:

- `DEBUG`
- `INFO`
- `WARNING`
- `ERROR`
- `CRITICAL`

#### Write to Syslog or Stdout
For each log event triggered, the output can write to the **syslog**, the **stdout**, or both. The accepted values are a boolean. When true, the log writes in the selected target:

- `"syslog": true`
- `"stdout": true`

#### Add a prefix to all lines
Besides, you might want to choose to add a string to every logged line, so you can quickly filter messages with external tools later.

- `"prefix": "[ANY STRING]"`

#### Predefined and customs formats
If you want to follow other patterns for logging, you're able to.

- `"format": "custom"`

The valid formats are:
 - `default`
 - `logstash`
 - `custom`

If you select the `custom` format, you'll be able to use the `custom_format` field:

- `"custom_format": "%{message}"`

The pattern to use is the same as the [go-logging library](https://github.com/op/go-logging/blob/master/format.go#L156)

## Logstash
If you want to log using the Logstash standard via stdout, you have to add the `krakend-logstash` integration in the
root level of your `krakend.json`, inside the `extra_config` section. **The `gologging` needs to be enabled too (but the format should be `default`)**.

For instance:

    "extra_config": {
      "telemetry/logstash": {
        "enabled": true
      },
      "telemetry/logging": {
        "level": "INFO",
        "prefix": "[KRAKEND]",
        "syslog": false,
        "stdout": true
      }
    }
