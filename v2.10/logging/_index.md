---
lastmod: 2025-05-16
old_version: true
date: 2018-10-30
toc: true
linktitle: Logging overview
title: Logging
description: Learn how to implement Syslog and Stdout logging in KrakenD API Gateway, enabling effective monitoring and troubleshooting of your API gateway and microservices
weight: 300
menu:
  community_v2.10:
    parent: "160 Monitoring, Logs, and Analytics"
meta:
  source: https://github.com/krakend/krakend-gologging
  namespace:
  - telemetry/logging
  scope:
  - service
---
The logging component is an essential configuration block for any installation that lets you choose **where** and **how** to log the gateway activity. It also opens the door to integrating other components for more advanced usage.

When you add the logging component, you can customize the format of the logs and send them both to the *stdout* and the *syslog*. However, if you don't use this component, then KrakenD uses the basic capabilities of [Lura](https://luraproject.org) (standard output only and a `DEBUG` level).

The `telemetry/logging` has the following logging capabilities:

- Write to the **Stdout (the console)**
- Write to the **Syslog (local file or remote server)**
- Add a prefix to log lines
- Select the reporting level
- Option to use a predefined or custom format
- Use custom layouts and define logged fields ([{{< badge >}}Enterprise{{< /badge >}} version](/docs/enterprise/logging/))

## Types of log messages
The content that KrakenD writes in its log represents two types of logging:

- **Access log**
- **Application log**

The access log and the application log can have different formats. For instance, you can have one in text plain and the other in JSON if needed. When you do this, you must parse them accordingly in your log ingestion.

### Access log
The access log shows **users' activity** and prints: which endpoints are requested, when, the status code, the duration, the requesting IP, and the method. The default format of the access log is as follows:

```log
[GIN] yyyy/mm/dd - hh:mm:ss | 200 |       4.529µs |      172.17.0.1 | GET      "/user/foo"
[GIN] yyyy/mm/dd - hh:mm:ss | 200 |       3.647µs |      172.17.0.1 | GET      "/category/bar"
```

**Access logs are never written in the syslog**, regardless of their configuration, and they show only in **stdout**.

While the access log is not customizable in the Open Source edition, {{< badge >}}Enterprise{{< /badge >}} allows you to write the format you want to output using the attribute `access_log_format` (see below).

#### Disabling the access log
You can also disable the access log setting the flag `disable_access_log`, or you can [remove specific requests from logs ](/docs/v2.10/service-settings/router-options/#remove-requests-from-logs), like when you don't want to see the health checks.


### Application log
The application log messages are the errors, warnings, debugging information, and other messages shown by the gateway while it operates.

The application logs are customizable as you can extend the functionality, such as sending the events to the **syslog**, using JSON format, choosing the verbosity level, or using the [Graylog Extended Log Format](/docs/v2.10/logging/graylog-gelf/) (GELF).

In addition to this, a lot of **exporters** are available to send your logs out to third parties (see [Telemetry](/docs/v2.10/telemetry/))

Application logs might look different on each application, but this is an example:

```
yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [SERVICE: Gin] Debug enabled
yyyy/mm/dd hh:mm:ss KRAKEND INFO: Starting the KrakenD instance
yyyy/mm/dd hh:mm:ss KRAKEND INFO: [SERVICE: Gin] Building the router
yyyy/mm/dd hh:mm:ss KRAKEND INFO: [SERVICE: Gin] Listening on port: 8080
yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [SERVICE: AsyncAgent][mkt-event] Starting the async agent
yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [ENDPOINT: mkt-event] Building the proxy pipe
yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [BACKEND: /__debug/some] Building the backend pipe
yyyy/mm/dd hh:mm:ss KRAKEND INFO: [SERVICE: AsyncAgent][AMQP][mkt-event] Starting the consumer
yyyy/mm/dd hh:mm:ss KRAKEND ERROR: [SERVICE: Asyncagent][mkt-event] building the amqp subscriber: dial tcp 192.168.2.223:5672: connect: connection refused
```

## Logging Configuration
To add ample logging capabilities, you need to add the component at the service level of your `krakend.json` configuration under the `extra_config` key:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "INFO",
      "prefix": "[KRAKEND]",
      "syslog": false,
      "stdout": true
    }
  }
}
```
These are the different supported configuration options:

{{< schema version="v2.10" data="telemetry/logging.json" >}}

### Customizing the application log
The attribute `format` allows you to set a formatter for the application log. The following values are available:

- `default`: Uses the pattern `%{time:2006/01/02 - 15:04:05.000} %{color}▶ %{level:.6s}%{color:reset} %{message}`
- `logstash`: **Logs in JSON format** using the logstash format. See [Logstash](/docs/v2.10/logging/logstash/) for more information. E.g.: `{"@timestamp":"%{time:2006-01-02T15:04:05.000+00:00}", "@version": 1, "level": "%{level}", "message": "%{message}", "module": "%{module}"}`.
- `custom`: Write the pattern from scratch, as defined in the `custom_format` attribute.

#### Variables available under custom_format
You can use these variables when defining the `custom_format`:

- `%{id}`: Prints the sequence number for log message (uint64).
- `%{pid}`: Prints the process id (int)
- `%{time}`: Prints the time when log occurred. It uses the [Go time format](https://pkg.go.dev/time). For instance `%{time:2006/01/02 - 15:04:05.000}`
- `%{level}`: Prints the log level
- `%{program}`: Prints the command running
- `%{message}`: Prints the application log message
- `%{module}`: Prints the module
- `%{color}`: Prints the ANSI color based on log level, the output can be adjusted to either use bold colors, e.g, `%{color:bold}` or to reset the ANSI attributes `%{color:reset}`.

For example, you can customize your pattern like this:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "INFO",
      "prefix": "[KRAKEND]",
      "syslog": false,
      "stdout": true,
      "format": "custom",
      "custom_format": "[MY APP LOG] %{time:2006/01/02 - 15:04:05.000} %{color}▶ %{level:.6s}%{color:reset} %{message}"
    }
  }
}
```
### Customizing the access log
Similarly, only on {{< badge >}}Enterprise{{< /badge >}} you can customize how the access log prints. The following `access_log_format` values are available:

- `default`: Uses `%{prefix} %{time} [AccessLog] |%{statusCode}| %{latencyMs} | %{clientIP} | %{method} %{path}\n` as pattern.
- `httpdCommon`: Uses `%{clientIP} - - [%{time}] \"%{method} %{path} %{proto}\" %{statusCode} -\n` as in the Apache HTTPd log format
- `httpdCombined`: The Apache HTTPd Combined log format `%{clientIP} - - [%{time}] \"%{method} %{path} %{proto}\" %{statusCode} - \"%{header.Referer}\" \"%{header.User-Agent}\"\n`
- `json`: Uses `{\"prefix\":\"%{prefix}\", \"time\":\"%{time}\", \"status_code\":%{statusCode}, \"latency\":\"%{latency}\", \"client_ip\":\"%{clientIP}\", \"method\":\"%{method}\", \"path\":\"%{path}\"}\n`
- `custom`: Write your own pattern, as defined in the `access_log_custom_format` attribute.


#### Variables available to `access_log_custom_format`
When the `access_log_format` is set to `custom`, you can use these variables under `access_log_custom_format` to specify your format:

- `%{prefix}`: The value you have set under the `prefix` attribute.
- `%{time}`: The time when the the access finished. The layout prints a format like 2006/01/02 - 15:04:05.000
- `%{statusCode}`: The response status code as given to the consumer
- `%{latencyMs}`: The operation latency in milliseconds with 3 decimals (microsecond resolution). This computes the time of the request from beginning to end.
- `%{latency}`: The operation latency in seconds with 3 decimals
- `%{clientIP}`: The real IP of the client
- `%{method}`: The HTTP verb used
- `%{path}`: The endpoint path
- `%{host}`: The host of the URL
- `%{header.xxx}`: The value of a specific header, where `xxx` is the header name.
- `%{scheme}`: The scheme used (e.g., http, https, ws)
- `%{jwt.xxx}`: The value of a specific claim in the token, where `xxx` is the claim name (only first level, non-nested, claims).
- `%{query}`: The query strings passed in the request
- `%{proto}`: The protocol used (e.g., HTTP/1.0, HTTP/2, etc)

For instance, you could **print the access log in JSON format** as follows:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "INFO",
      "prefix": "[KRAKEND]",
      "syslog": false,
      "stdout": true,
      "access_log_format": "json"
    }
  }
}
```
Or you could have a log that includes the JWT subject, the authorization header and query strings as follows:

```json
{
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "INFO",
      "prefix": "[KRAKEND]",
      "syslog": false,
      "stdout": true,
      "access_log_format": "custom",
      "access_log_custom_format": "[AccessLog] %{prefix} %{time} | %{statusCode} | %{latencyMs} | %{clientIP} | %{method} %{scheme}://%{host}%{path}?%{query} %{query.bar} %{header.Authorization} %{jwt.sub}\n"
    }
  }
}
```

## Writing the log on a file
Although logging on disk might impact software performance and is discouraged in high-throughput systems, you can still store the logs in a file.

**Avoid redirecting the output** (e.g.: `krakend run > krakend.log`) and **use the *syslog* of your machine instead**.

To setup logs on disk, you should consider the following steps:

1) Add the syslog configuration to yor `krakend.json`
2) Add a specific entry for krakend under `/etc/rsyslog.d/`
3) Optionally add log rotation

### 1. Syslog configuration
```json
{
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "WARNING",
      "syslog": true,
      "stdout": true
    }
  }
}
```

You might set the `stdout` to `false` if you don't want to check on the console but only on the logs.

### 2. Add an entry to `rsyslog`
The folder `/etc/rsyslog.d/` shows the different configurations of the system. We will create a new file `/etc/rsyslog.d/krakend.conf` and place this content inside:

    local3.*    -/var/log/krakend.log

If you are familiar with *syslog*, you change the `syslog_facility` to any other (local) value and adjust it in the file above.

### 3. KrakenD log rotation
The syslog will take care of populating the log and can be used conveniently with the default system tools like **rotating the logs** with `logrotate`. Add a new configuration file `/logrotate.d/krakend` and add the content below:

```
/var/log/krakend.log {
  rotate 7
  daily
  missingok
  delaycompress
  compress
  postrotate
    /usr/lib/rsyslog/rsyslog-rotate
  endscript
}
```
