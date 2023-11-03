---
lastmod: 2022-10-25
date: 2018-10-30
toc: true
linktitle: Standard Logging
title: Logging in KrakenD API Gateway
description: Learn how to implement Syslog and Stdout logging in KrakenD API Gateway, enabling effective monitoring and troubleshooting of your API gateway and microservices
weight: 10
aliases: ["/docs/logging-metrics-tracing/logging/", "/docs/logging/extended-logging/"]
menu:
  community_current:
    parent: "090 Logging"
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

## Types of log messages
The content that KrakenD writes in its log represents two types of logging:

- **Access logs**
- **Application logs**

### Access logs
The access log shows users' activity and prints: which endpoints are requested, when, the status code, the duration, the requesting IP, and the method. An example of the format is:

```log
[GIN] yyyy/mm/dd - hh:mm:ss | 200 |       4.529µs |      172.17.0.1 | GET      "/user/foo"
[GIN] yyyy/mm/dd - hh:mm:ss | 200 |       3.647µs |      172.17.0.1 | GET      "/category/bar"
```

The access log content is not customizable. It can be disabled using `disable_access_log` (see how to [remove requests from logs options](/docs/service-settings/router-options/#remove-requests-from-logs)).

**Access logs are never written in the syslog**, regardless of their configuration, and they show only in **stdout**.

### Application logs
The application log messages are the errors, warnings, debugging information, and other messages shown by the gateway while it operates.

The application logs are customizable as you can extend the functionality, such as sending the events to the **syslog**, using JSON format, choosing the verbosity level, or using the [Graylog Extended Log Format](/docs/logging/graylog-gelf/) (GELF).

In addition to this, a lot of **exporters** are available to send your logs out to third parties (see [Telemetry](/docs/telemetry/))

Application logs might look different on each application, but this is an example:

```log
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

{{< schema data="telemetry/logging.json" >}}

When setting a predefined `format` the output is:

- `default`: Uses the pattern `%{time:2006/01/02 - 15:04:05.000} %{color}▶ %{level:.6s}%{color:reset} %{message}`
- `logstash`: **Logs in JSON format** using the logstash format. See [Logstash](/docs/logging/logstash/) for more information. E.g.: `{"@timestamp":"%{time:2006-01-02T15:04:05.000+00:00}", "@version": 1, "level": "%{level}", "message": "%{message}", "module": "%{module}"}`.
- `custom`: You set the format using `custom_format`. To know more about the possible **patterns** see the [go-logging library](https://github.com/op/go-logging/blob/master/format.go#L156)


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
