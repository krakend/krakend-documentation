---
lastmod: 2022-06-15
old_version: true
date: 2018-10-30
toc: true
linktitle: Improved logging
title: Improved Logging - Syslog, stdout
weight: 10
source: https://github.com/krakend/krakend-gologging
menu:
  community_v2.0:
    parent: "090 Logging"
---
By default,  when KrakenD starts all the log events are sent to the **standard output** using the basic logger capabilities of the [Lura Project](https://luraproject.org). The reporting level, in that case, is `DEBUG` and adds no prefix to the log lines.

There are two types of logs:

- **Access logs** (activity from users)
- **Application logs** (errors, debugging information and other application messages)

## Access logs
The access log shows with the following format:

    [GIN] 2022/06/15 - 15:38:58 | 200 |       4.529µs |      172.17.0.1 | GET      "/test/foo"
    [GIN] 2022/06/15 - 15:38:59 | 200 |       3.647µs |      172.17.0.1 | GET      "/test/bar"

The access log is not customizable, but it can be disabled using `disable_access_log` (see how to [remove requests from logs options](/docs/v2.0/service-settings/router-options/#remove-requests-from-logs)).

## Application logs

Different logging components allow you to extend the application logging functionality, such as sending the events to the **syslog**, use JSON format, choosing the verbosity level, or use the **Graylog Extended Log Format (GELF)**.

In addition to this, a lot of exporters are available to send your logs out (see [Telemetry](/docs/v2.0/telemetry/))

The component `telemetry/logging` extends the default logging capabilities with the following capabilities:

- Option to write to the stdout
- Option to write to the syslog
- Add a prefix to log lines
- Select the reporting level
- Option to use a predefined or custom format

To enjoy the extended logging capabilities the component needs to be added in the `krakend.json` configuration. Add its namespace in the `extra_config` at the root level:

{{< highlight json >}}
{
  "version": 3,
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
}
{{< /highlight >}}


The snippet above shows the four options you can configure, explained below.

### Set the *reporting level*
Define the severity you would like to see in the logs with `level`. The recognized options from more to less verbosity are:

- `DEBUG`
- `INFO`
- `WARNING`
- `ERROR`
- `CRITICAL`

### Write to Syslog or Stdout
For each log event triggered, the output can write to the **syslog**, the **stdout**, or both. The accepted values are a boolean. When true, the log writes in the selected target:

- `"syslog": true`
- `"stdout": true`

### Add a prefix to all lines
Besides, you might want to choose to add a string to every logged line, so you can quickly filter messages with external tools later.

- `"prefix": "[ANY STRING]"`

### Predefined and customs formats
If you want to follow other patterns for logging, you're able to.

- `"format": "custom"`

The valid formats are:
 - `default` uses the pattern `%{time:2006/01/02 - 15:04:05.000} %{color}▶ %{level:.6s}%{color:reset} %{message}`
 - `logstash` uses the pattern `{"@timestamp":"%{time:2006-01-02T15:04:05.000+00:00}", "@version": 1, "level": "%{level}", "message": "%{message}", "module": "%{module}"}`. See [Logstash](/docs/v2.0/logging/logstash/)
 - `custom` lets you write your own pattern, e.g: `%{message}`

To know more about the possible pattern format see the [go-logging library](https://github.com/op/go-logging/blob/master/format.go#L156)

## Log in JSON format
If you want to log in the stdout using **JSON format** see [Logstash](/docs/v2.0/logging/logstash/)