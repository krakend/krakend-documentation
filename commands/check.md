---
aliases: ["/commands/check"]
lastmod: 2021-10-25
date: 2016-10-28
linktitle: Checking the config
title: Validating the configuration with `check`
description: KrakenD Check Command
weight: 10
menu:
  community_current:
    parent: "020 Command Line"
---

The `krakend check` command **validates KrakenD configuration files** written in any of its [supported formats](/docs/configuration/supported-formats/). The validation does two different things: **linting** and **testing** (an execution).

Whenever the `check` is run, it acts as a (JSON) linter and makes sure that the configuration is not broken from a syntax point of view. It also works with [flexible configuration](/docs/configuration/flexible-config/). When used with the test flag `-t` it also tries to test a run of the service to catch problems that are not strictly related to linting but runtime. For instance, you could have declared a colliding or duplicated endpoint: the syntax would valid and properly linted yet the configuration impossible to run. 

{{< note title="KrakenD doesn't do strict parsing" >}}
KrakenD **ignores all unknown keys**, and it does not warn about it. It allows you to declare new parameters and entries, but it can also shadow errors in the configuration. The `--debug` flag (see below) shows what is parsed and processed.
{{< /note >}}

The `krakend check` command accepts the following options:

{{< terminal title="Usage of KrakenD check" >}}
./krakend check -h
                                                                        
`7MMF' `YMM'                  `7MM                         `7MM"""Yb.   
  MM   .M'                      MM                           MM    `Yb. 
  MM .d"     `7Mb,od8 ,6"Yb.    MM  ,MP'.gP"Ya `7MMpMMMb.    MM     `Mb 
  MMMMM.       MM' "'8)   MM    MM ;Y  ,M'   Yb  MM    MM    MM      MM 
  MM  VMA      MM     ,pm9MM    MM;Mm  8M""""""  MM    MM    MM     ,MP 
  MM   `MM.    MM    8M   MM    MM `Mb.YM.    ,  MM    MM    MM    ,dP' 
.JMML.   MMb..JMML.  `Moo9^Yo..JMML. YA.`Mbmmd'.JMML  JMML..JMMmmmdP'   
_______________________________________________________________________
                                                                  
Version: {{< version >}}

Validates that the active configuration file has a valid syntax to run the service.
Change the configuration file by using the --config flag

Usage:
  krakend check [flags]

Aliases:
  check, validate

Examples:
krakend check -d -c config.json

Flags:
  -h, --help              help for check
  -i, --indent string     Indentation of the check dump (default "\t")
  -t, --test-gin-routes   Test the endpoint patterns against a real gin router on selected port

Global Flags:
  -c, --config string   Path to the configuration filename
  -d, --debug count     Enable the debug
{{< /terminal >}}

## Flags
Use `krakend check` in combination with the following flags:

- `-c` or `--config` (mandatory) to specify the path to the configuration file in any of the [supported formats](/docs/configuration/supported-formats/), or to the starting template if used in combination with flexible configuration. 
- `-d` or `--debug` to enable the debug and see information about how KrakenD is interpreting your configuration file. Use from 1 to 3 levels of verbosity using `-d`, `-dd`, or `-ddd`.
- Use `-t` to **test** the configuration and try to start the service with it for a second. This option is highly recommended as prevents conflicting routes and other problems that are not related with the linting itself, and would end up in a *panic*.
- `-i` or `--indent`, in combination with `-d`, to change the identation when the debug information renders (default: tab). E.g.: `-i "#"` uses a hash instead of a tab for every nesting level.

## Add `check` to your CI/CD pipeline
The check command is a must in any **CI/CD pipeline** or when building an immutable `Dockerfile`, as it lets you find broken configurations before going live. Consider adding a line like the following in your release process:

{{< terminal title="Recommended file check for CI/CD" >}}
krakend check -t -d -c path/to/krakend.json
{{< /terminal >}}

### Docker image with check and flexible configuration
The following example illustrates a combination of the `check` command with a a multi-stage build to compile a [flexible configuration](/docs/configuration/flexible-config/) in a `Dockerfile`:

{{< highlight docker >}}
FROM devopsfaith/krakend as builder
ARG ENV=prod

COPY krakend.tmpl .
COPY config .

# Save temporary file to /tmp to avoid permission errors
RUN FC_ENABLE=1 \
    FC_OUT=/tmp/krakend.json \
    FC_PARTIALS="/etc/krakend/partials" \
    FC_SETTINGS="/etc/krakend/settings/$ENV" \
    FC_TEMPLATES="/etc/krakend/templates" \
    krakend check -d -t -c krakend.tmpl

FROM devopsfaith/krakend
COPY --from=builder --chown=krakend /tmp/krakend.json .
{{< /highlight >}}

Notice how the lines above `check` that the configuration is valid using a starting template `krakend.tmpl` and output the compiled template into a `/tmp/krakend.json` file. This file is the only addition to the final Docker image.

It assumes that you have a file structure like this:

	.
	├── config
	│   ├── partials
	│   ├── settings
	│   │   ├── prod
	│   │   │   └── env.json
	│   │   └── test
	│   │       └── env.json
	│   └── templates
	│       └── some.tmpl
	├── Dockerfile
	└── krakend.tmpl

And that you build with something like:

{{< terminal title="Docker build" >}}
docker build --build-arg ENV=prod -t krakend-prod .
{{< /terminal >}}


## Debugging your configuration
There are 3 different levels of verbosity you can use with the `--debug` (or `-d`) flag. When used a single time you get the most relevant information after parsing the configuration. The following example shows the debu of a configuration with one endpoint:

{{< terminal title="Checking the configuration with the debug flag" >}}
krakend check -t -d -c krakend.json
Parsing configuration file: krakend.json
Parsing configuration file: krakend.json
Global settings
    Name: My lovely gateway
    Port: 8080
4 global component configuration(s):
- github_com/devopsfaith/krakend-botdetector
- github_com/devopsfaith/krakend-cors
- github_com/devopsfaith/krakend-gologging
- github_com/devopsfaith/krakend-metrics
1 API endpoints:
    - GET /cel/req-resp/:id
    Timeout: 3s
    1 endpoint component configuration(s):
    - github.com/devopsfaith/krakend-cel
    Connecting to 2 backend(s):
        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        1 backend component configuration(s):
        - github.com/devopsfaith/krakend-cel

        [+] GET /__debug/{{.Id}}
        Timeout: 3s
        Hosts: [http://127.0.0.1:8080]
        1 backend component configuration(s):
        - github.com/devopsfaith/krakend-cel

Syntax OK!
{{< /terminal >}}

The same example with a verbosity of level 2 (`-dd`), adds more information in the global settings (like the TLS section) and shows the configuration of the extra_config. The endpoints and the backends show also more information:

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
- github_com/devopsfaith/krakend-botdetector
    deny: [a b]
    patterns: [(Pingdom.com_bot_version_).* (facebookexternalhit)/.*]
    allow: [c Pingdom.com_bot_version_1.1]
- github_com/devopsfaith/krakend-cors
    expose_headers: [Content-Length]
    allow_origins: [*]
    max_age: 12h
    allow_methods: [POST GET]
    allow_headers: [Origin Authorization Content-Type]
- github_com/devopsfaith/krakend-gologging
    prefix: [KRAKEND]
    stdout: true
    level: DEBUG
    syslog: false
- github_com/devopsfaith/krakend-metrics
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
    - github.com/devopsfaith/krakend-cel
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
        - github.com/devopsfaith/krakend-cel
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
        - github.com/devopsfaith/krakend-cel
            [map[check_expr:int(req_params.Id) % 5 == 0]]

Syntax OK!
{{< /terminal >}}

And in the level 3 (`-ddd`) there is everything that KrakenD could parse from the configuration:

{{< terminal title="Checking the configuration with the debug flag" >}}
krakend check -t -ddd -c krakend.json
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
- github_com/devopsfaith/krakend-botdetector
    patterns: [(Pingdom.com_bot_version_).* (facebookexternalhit)/.*]
    deny: [a b]
    allow: [c Pingdom.com_bot_version_1.1]
- github_com/devopsfaith/krakend-cors
    expose_headers: [Content-Length]
    allow_methods: [POST GET]
    allow_origins: [*]
    allow_headers: [Origin Authorization Content-Type]
    max_age: 12h
- github_com/devopsfaith/krakend-gologging
    level: DEBUG
    syslog: false
    stdout: true
    prefix: [KRAKEND]
- github_com/devopsfaith/krakend-metrics
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
    - github.com/devopsfaith/krakend-cel
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
        - github.com/devopsfaith/krakend-cel
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
        - github.com/devopsfaith/krakend-cel
            [map[check_expr:int(req_params.Id) % 5 == 0]]

Syntax OK!
{{< /terminal >}}