---
lastmod: 2021-03-01
old_version: true
date: 2016-10-28
linktitle: Validating the config
title: Validating the configuration with `check`
description: KrakenD Check Command
weight: 20
notoc: true
menu:
  community_v2.0:
    parent: "010 Configuration file(s)"
---

The `krakend check` command **validates KrakenD configuration files** written in any of its [supported formats](/docs/v2.0/configuration/supported-formats/).

It's able to perform three things:

- **Syntax validation** - For any format (`.yml`, `.json`, `.toml`, etc)
- **Linting** - Besides checking that the file isn't malformed, the linter checks your config exhaustively against KrakenD's [official schema](https://github.com/krakendio/krakend-schema) to detect wrong types, unknown attributes, or misplaced components. Only available when you work with `JSON` formats.
- **Testing** - It tests a run of the service to catch problems that are not strictly related to linting but to the runtime. For instance, you could declare a colliding endpoint, and the syntax would validate and lint, yet the configuration would be impossible to run.

**The `check` command can guarantee that a configuration is valid with the three validations**.

**TL;DR**: Add the following line before deploying:

{{< terminal title="Term" >}}
krakend check -tlc krakend.json
{{< /terminal >}}

See the usage below.

## Checking the configuration
The `krakend check` command accepts the following options:

{{< terminal title="Usage of KrakenD check" >}}
./krakend check -h
{{< ascii-logo >}}

Version: v2.0

Validates that the active configuration file has a valid syntax to run the service.
Change the configuration file by using the --config flag

Usage:
  krakend check [flags]

Aliases:
  check, validate

Examples:
krakend check -d -c config.json

Flags:
  -c, --config string     Path to the configuration filename
  -d, --debug count       Enables the debug
  -h, --help              help for check
  -i, --indent string     Indentation of the check dump (default "\t")
  -l, --lint              Enables the linting against the official KrakenD JSON schema
  -t, --test-gin-routes   Test the endpoint patterns against a real gin router on selected port
{{< /terminal >}}

## Flags
Use `krakend check` in combination with the following flags:

- `-c` or `--config` to specify the path to the configuration file in any of the [supported formats](/docs/v2.0/configuration/supported-formats/), or to the starting template if used in combination with flexible configuration.
- `-d` or `--debug` (*optional*) to enable the debug and see information about how KrakenD is interpreting your configuration file. Use from 1 to 3 levels of verbosity using `-d`, `-dd`, or `-ddd`.
- `-t` or `--test-gin-routes` (*optional*) to test the configuration by trying to start the service for a second. This option is highly recommended as it prevents conflicting routes and other problems unrelated to the linting itself and would end up in a *panic*.
- `-l` or `--lint` (*optional*) to check that your configuration file is properly linted and does not contain unrecognized options or wrong types. **Your configuration must be in JSON format**. When using Flexible Configuration, you cannot check directly the .`krakend.tmpl` file, you need to do it in a second round using the content of the `FC_OUT` file. This option requires Internet access as the schema is published online under `https://www.krakend.io/schema/krakend.json`.
- `-i` or `--indent` (*optional*) in combination with `-d`, to change the indentation when the debug information renders (default: `TAB`). E.g.: `-i "#" ` uses a hash instead of a tab for every nesting level.

{{< note title="Use --lint to do strict parsing" >}}
The command `krakend run` will run any syntax-valid file, **ignoring unknown configuration keys**. Use the `--lint` flag in the check command to find incorrect entries.
{{< /note >}}

## Debugging your configuration
You can use three different verbosity levels with the `--debug` (or `-d') flag. The levels are `-d`, `-dd`, and `-ddd`. When used a single time, you get the most relevant information after parsing the configuration, when you add more, you get more and more details. The following example shows the debug of a configuration with one endpoint:

{{< terminal title="Checking the configuration with the debug flag" >}}
krakend check -t --lint -d -c krakend.json
Parsing configuration file: krakend.json
Global settings
    Name: My lovely gateway
    Port: 8080
4 global component configuration(s):
- security/bot-detector
- security/cors
- telemetry/logging
- telemetry/metrics
1 API endpoints:
    - GET /cel/req-resp/:id
    Timeout: 3s
    1 endpoint component configuration(s):
    - validation/cel
    Connecting to 2 backend(s):
        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        1 backend component configuration(s):
        - validation/cel

        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        1 backend component configuration(s):
        - validation/cel
1 async agent(s):
    - cool-agent
    1 agent component configuration(s):
    - github.com/devopsfaith/krakend-amqp/agent
    Connecting to 1 backend(s):
        [+] POST /__debug/cool-agent
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]

Syntax OK!
{{< /terminal >}}

The same example with the verbosity level 2 (`-dd`) adds more information in the global settings (like the TLS section) and shows the configuration of the extra_config. The endpoints and the backends also show more information:

{{< terminal title="Checking the configuration with the debug flag" >}}
Parsing configuration file: krakend.json
Global settings
    Name: My lovely gateway
    Port: 8080
    Default cache TTL: 1h0m0s
    Default timeout: 3s
    Default backend hosts: []
    No TLS section defined
    No Plugin section defined
4 global component configuration(s):
- security/bot-detector
    deny: [a b]
    patterns: [(Pingdom.com_bot_version_).* (facebookexternalhit)/.*]
    allow: [c Pingdom.com_bot_version_1.1]
- security/cors
    expose_headers: [Content-Length]
    allow_origins: [*]
    max_age: 12h
    allow_methods: [POST GET]
    allow_headers: [Origin Authorization Content-Type]
- telemetry/logging
    prefix: [KRAKEND]
    stdout: true
    level: DEBUG
    syslog: false
- telemetry/metrics
    listen_address: :8090
    collection_time: 60s
1 API endpoints:
    - GET /example/:id
    Timeout: 3s
    QueryString: [q id]
    CacheTTL: 1h0m0s
    Headers to pass: [X-Header]
    OutputEncoding: json
    Concurrent calls: 1
    1 endpoint component configuration(s):
    - validation/cel
        [map[check_expr:'something' in req_headers['X-Header']] map[check_expr:int(req_params.Id) % 2 == 0]]
    Connecting to 2 backend(s):
        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        Concurrent calls: 1
        Host sanitization disabled: false
        Target:
        Deny: [], Allow: []
        Mapping: map[]
        Group: backend1
        Encoding:
        Is collection: false
        SD:
        1 backend component configuration(s):
        - validation/cel
            [map[check_expr:int(req_params.Id) % 3 == 0]]

        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        Concurrent calls: 1
        Host sanitization disabled: false
        Target:
        Deny: [], Allow: []
        Mapping: map[]
        Group: backend2
        Encoding:
        Is collection: false
        SD:
        1 backend component configuration(s):
        - validation/cel
            [map[check_expr:int(req_params.Id) % 5 == 0]]
1 async agent(s):
    - cool-agent
    Encoding:
    Consumer Timeout: 3s
    Consumer Workers: 20
    Consumer Topic: *.bar
    Consumer Max Rate: 0.000000
    Connection Max Retries: 10
    Connection Backoff Strategy: exponential-jitter
    Connection Health Interval: 1s
    1 agent component configuration(s):
    - github.com/devopsfaith/krakend-amqp/agent
        name: krakend
        host: amqp://guest:guest@localhost:5672/
        exchange: foo
        prefetch_count: 40
        auto_ack: true
    Connecting to 1 backend(s):
        [+] POST /__debug/cool-agent
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        Concurrent calls: 0
        Host sanitization disabled: false
        Target:
        Deny: [], Allow: []
        Mapping: map[]
        Group:
        Encoding:
        Is collection: false
        SD:
        0 backend component configuration(s):

Syntax OK!
{{< /terminal >}}

And in level 3 (`-ddd`), there is everything that KrakenD could parse from the configuration:

{{< terminal title="Checking the configuration with the debug flag" >}}
krakend check -t --lint -ddd -c krakend.json
Parsing configuration file: krakend.json
Global settings
    Name: My lovely gateway
    Port: 8080
    Default cache TTL: 1h0m0s
    Default timeout: 3s
    Default backend hosts: []
    Read timeout: 0s
    Write timeout: 0s
    Idle timeout: 0s
    Read header timeout: 0s
    Idle connection timeout: 0s
    Response header timeout: 0s
    Expect continue timeout: 0s
    Dialer timeout: 0s
    Dialer fallback delay: 0s
    Dialer keep alive: 0s
    Disable keep alives: false
    Disable compression: false
    Max idle connections: 0
    Max idle connections per host: 250
    No TLS section defined
    No Plugin section defined
4 global component configuration(s):
- security/bot-detector
    patterns: [(Pingdom.com_bot_version_).* (facebookexternalhit)/.*]
    deny: [a b]
    allow: [c Pingdom.com_bot_version_1.1]
- security/cors
    expose_headers: [Content-Length]
    allow_methods: [POST GET]
    allow_origins: [*]
    allow_headers: [Origin Authorization Content-Type]
    max_age: 12h
- telemetry/logging
    level: DEBUG
    syslog: false
    stdout: true
    prefix: [KRAKEND]
- telemetry/metrics
    listen_address: :8090
    collection_time: 60s
1 API endpoints:
    - GET /example/:id
    Timeout: 3s
    QueryString: [q id]
    CacheTTL: 1h0m0s
    Headers to pass: [X-Header]
    OutputEncoding: json
    Concurrent calls: 1
    1 endpoint component configuration(s):
    - validation/cel
        [map[check_expr:'something' in req_headers['X-Header']] map[check_expr:int(req_params.Id) % 2 == 0]]
    Connecting to 2 backend(s):
        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        Concurrent calls: 1
        Host sanitization disabled: false
        Target:
        Deny: [], Allow: []
        Mapping: map[]
        Group: backend1
        Encoding:
        Is collection: false
        SD:
        1 backend component configuration(s):
        - validation/cel
            [map[check_expr:int(req_params.Id) % 3 == 0]]

        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        Concurrent calls: 1
        Host sanitization disabled: false
        Target:
        Deny: [], Allow: []
        Mapping: map[]
        Group: backend2
        Encoding:
        Is collection: false
        SD:
        1 backend component configuration(s):
        - validation/cel
            [map[check_expr:int(req_params.Id) % 5 == 0]]

1 async agent(s):
    - cool-agent
    Encoding:
    Consumer Timeout: 3s
    Consumer Workers: 20
    Consumer Topic: *.bar
    Consumer Max Rate: 0.000000
    Connection Max Retries: 10
    Connection Backoff Strategy: exponential-jitter
    Connection Health Interval: 1s
    1 agent component configuration(s):
    - github.com/devopsfaith/krakend-amqp/agent
        exchange: foo
        prefetch_count: 40
        auto_ack: true
        name: krakend
        host: amqp://guest:guest@localhost:5672/
    Connecting to 1 backend(s):
        [+] POST /__debug/cool-agent
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        Concurrent calls: 0
        Host sanitization disabled: false
        Target:
        Deny: [], Allow: []
        Mapping: map[]
        Group:
        Encoding:
        Is collection: false
        SD:
        0 backend component configuration(s):

Syntax OK!
{{< /terminal >}}