---
lastmod: 2020-06-13
old_version: true
date: 2018-09-21
linktitle: Multiple configuration files
source: https://github.com/devopsfaith/krakend-flexibleconfig
since: 0.5
menu:
  community_v1.4:
    parent: "010 Configuration file(s)"
images:
- /images/krakend-flexible-config.png
title: Flexible configuration
weight: 20
---
The **Flexible Configuration** component is included in the KrakenD API Gateway and allows you to **split the configuration into multiple files** while **using variables** and **templates**.

The Flexible Configuration enables template processing. It compiles during start-up time. With this, you have the opportunity to produce a more sophisticated configuration file that utilizes variables and brings content from external files.

A template system gives you full flexibility to work with the configuration file. It comes handy to:

- Split a large `krakend.json` file into several pieces
- Inject variables, like environment settings, into the configuration
- Use placeholders and reusable code blocks to avoid repeating code
- Organize your code better when there are multiple developers modifying the gateway
- Track changes and review code more easily
- Have the full power of the go template  system!

## Requirements
The only requirement to use the Flexible Configuration is to encode the configuration file in `JSON` format as the package does not support other formats just yet.

## Usage
The activation of the package works via environment variables when running krakend, as follows:

- `FC_ENABLE=1` to let KrakenD know that you are using Flexible Configuration. You can use `1` or any other value (but  `0` won't disable it!). The file passed with the `-c` flag is the base template.
- `FC_SETTINGS=dirname`: The path to the directory with all the settings files.
- `FC_PARTIALS=dirname`: The path to the directory with the partial files included in the configuration file. Partial files DON'T EVALUATE, they are only inserted in the placeholder.
- `FC_TEMPLATES=dirname`: The path to the directory with the sub-templates included in the configuration file. These are evaluated using the Go templating system.
- `FC_OUT`: For debugging purposes, saves the resulting configuration of processing the flexible configuration in the given filename. Otherwise, the final file is not visible.

For instance, let's assume you decided to organize your configuration as follows:

    .
    └── config
       ├── krakend.json
       ├── partials
       │   └── static_file.tmpl
       ├── templates
       │   └── environment.tmpl
       └── settings
           ├── prod
           |   └── db.json
           └── dev
               └── db.json

Then you can run KrakenD from the terminal with this command:

{{< terminal title="Enabling flexible configuration with your custom dirs" >}}
FC_ENABLE=1 \
FC_SETTINGS="config/prod/settings" \
FC_PARTIALS="config/partials" \
FC_TEMPLATES="config/templates" \
krakend run -c "config/krakend.json"
{{< /terminal >}}

In the example above notice that the `FC_SETTINGS` includes the path to the `prod`uction folder. You might inject here an env var if you have multiple environments. The directory structure is completely up to you.

{{< note title="Consider adding an alias" >}}
Consider adding an alias or bash script with the command above to speed up your development time.
{{< /note >}}

### Template syntax
The configuration file passed with the `-c` flag is treated as a **Go template** ([documentation](https://golang.org/pkg/text/template/)), and you can make use of all the power the template engine brings. In addition, it also loads [Sprig functions](http://masterminds.github.io/sprig/). The data evaluations or control structures are easily recognized as they are surrounded by `{{` and `}}`. Any other text outside the delimiters is copied to the output unchanged.

These are all the syntax possibilities:

- `{{ .file.key }}`: Insert the value of a `key` in a settings `file`
- `{{ marshal .file.key }}`: Insert a JSON structure under a `key` in a settings `file`
- `{{ include "file.txt" }}`: Replace with the complete content of the `file.txt`
- `{{ template "file.tmpl" context }}`: Process the Go template `file.tmpl` passing in its dot (`{{ . }}`) the `context`

See below for further explanation and examples.

#### Insert values from settings files
In the `FC_SETTINGS` directory, you can save different `.json` files with data structures inside that you can reference in the templates.

For instance, if you have a file `db.json` with the following content:

    {
        "host": "192.168.1.23",
        "port": 8766,
        "pass": "a-p4ssw0rd",
        "label": "production"
    }

You can access particular settings like using this syntax: `{{ .db.host }}`.

The first name after the dot is the name of the file, and then the element in the structure you want to access. The example would write `192.168.1.23` where you wrote the placeholder.

{{< note title="Accessing variables inside a 'range'" >}}
When you are doing a loop with a `range`, the variables inside are relative to the `range` context. To access outsider variables use the `$` notation. For instance: `{{ $.db.host }}`
{{< /note >}}

#### Insert structures from settings files
When instead of a single value you need to insert a **JSON structure** (several elements), you need to use `marshal`.

    {{ marshal .db }}

The example would write the entire content of the `db.json` file.

#### Include an external file
To insert the content of an external partial file in-place use:

    {{ include "partial_file_name.txt" }}

**The content inside the partial template is not parsed**, and is inserted *as is* in plain text. The file is assumed to live inside the directory defined in `FC_PARTIALS` and can have any name and extension.

#### Include and process a sub-template
While the `include` is only meant to paste the content of a plain text file, the `template` gives you all the power of Go templating ([documentation](https://golang.org/pkg/text/template/)). The syntax is as follows:

    {{ template "template_name.tmpl" context }}

The template `template_name.tmpl` is executed and processed. The value of `context` is passed in the template as the context, meaning that the sub-template can access it using the dot `{{ . }}`. This context variable could be an object, such as `{{ template "environment" .db.label }}`, but it can also be another type, like a string: ``{{ template "environment" "production" }}`.

Go templates allow you to introduce handy stuff like conditionals or loops and allow you to create powerful configurations.

### Advanced functions
Complementing the [Go built-in template language](http://golang.org/pkg/text/template/), KrakenD adds to the Flexible Configuration more than 100 commonly used template functions using **Sprig**.

Some examples are:
- String Functions: `trim`, `wrap`, `randAlpha`, `plural` and more.
  - String List Functions: `splitList`, `sortAlpha` and more.
- Integer Math Functions: `add`, `max`, `mul` and more.
  - Integer Slice Functions: `until`, `untilStep`
- Date Functions: `now`, `date` and more.
- Defaults Functions: `default`, `empty`, `coalesce`, `toJson`, `toPrettyJson`, `ternary`
- Encoding Functions: `b64enc`, `b64dec` and more.
- Lists and List Functions: `list`, `first`, `uniq` and more.
- Dictionaries and Dict Functions: `set`, `dict`, `hasKey`, `pluck`, `deepCopy` and more.
- Type Conversion Functions: `atoi`, `int64`, `toString` and more.
- Path and Filepath Functions: `base`, `dir`, `ext`, `clean`, `isAbs`
- Flow Control Functions: `fail`
- Advanced Functions
  - UUID Functions: `uuidv4`
  - OS Functions: `env`, `expandenv`
  - Version Comparison Functions: `semver`, `semverCompare`
  - Reflection: `typeOf`, `kindIs`, `typeIsLike` and more.
  - Cryptographic and Security Functions: `derivePassword`, `sha256sum`, `genPrivateKey` and more.
  - Network: `getHostByName`

For more details see the [Sprig documentation](http://masterminds.github.io/sprig/)

### Testing the configuration
As the configuration is now composed of several pieces, it's easy to make a mistake at some point. Test the syntax of all the files is good with the `krakend check` command and pay attention to the output to verify there aren't any errors.

You might also want to use the flag `FC_OUT` to write the content of the final file in a known path, so you can check its contents:

    FC_ENABLE=1 \
    FC_SETTINGS="$PWD/config/settings" \
    FC_PARTIALS="$PWD/config/partials" \
    FC_TEMPLATES="$PWD/config/templates" \
    FC_OUT=out.json \
    krakend check -t -d -c "$PWD/config/krakend.json"

When there are errors, the output contains information to help you resolve it, e.g.:

    ERROR parsing the configuration file: loading flexible-config settings:
    - backends.json: invalid character '}' looking for beginning of object key string

## Flexible configuration example
To demonstrate the usage of the flexible configuration, we are going to reorganize a configuration file in several pieces. This is a simple example to see the basics of the templates system, and it does not intend to show a good way to organize and split your files:

    .
    └── config
        ├── krakend.json
        ├── partials
        │   └── rate_limit_backend.tmpl
        └── settings
            ├── endpoint.json
            └── service.json

**partials/rate_limit_backend.tmpl**

In this file, we have written the content of the rate limit configuration for a backend. This file is inserted when included "as is":

    "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
        "maxRate": "100",
        "capacity": "100"
    }

**settings/service.json**

In the settings directory, we write all the files whose values can be accessed as variables.

    {
        "port": 8090,
        "default_hosts": [
            "https://catalog-api-01.srv",
            "https://catalog-api-02.srv",
            "https://catalog-api-03.srv"
        ],
        "extra_config": {
            "github_com/devopsfaith/krakend-httpsecure": {
            "allowed_hosts": [],
            "ssl_proxy_headers": {
                "X-Forwarded-Proto": "https"
            },
            "ssl_certificate": "/opt/rsa.cert",
            "ssl_private_key": "/opt/rsa.key"
            }
        }
    }

**settings/endpoint.json**

This file declares a couple of endpoints that feed on a single backend:

    {
        "example_group": [
            {
                "endpoint": "/users/{id}",
                "backend": "/v1/users?userId={id}"
            },
            {
                "endpoint": "/posts/{id}",
                "backend": "/posts?postId={id}"
            }
        ]
    }

**krakend.json**

Finally, introducing the base template. It inserts the content of other files using `include`, uses the variables declared in the settings files and writes json content with `marshal`.

Have a look at the highlighted lines:

{{< highlight go-text-template "hl_lines=3-5 7 9 12 15" >}}
    {
        "version": 2,
        "port": {{ .service.port }},
        "extra_config": {{ marshal .service.extra_config }},
        "host": {{ marshal .service.default_hosts }},
        "endpoints": [
            {{ range $idx, $endpoint := .endpoint.example_group }}
            {{if $idx}},{{end}}
            {
            "endpoint": "{{ $endpoint.endpoint }}",
            "backend": [
                {
                    "url_pattern": "{{ $endpoint.backend }}",
                    "extra_config": {
                        {{ include "rate_limit_backend.tmpl" }}
                    }
                }
            ]
            {{ end }}
        ]
    }
{{< /highlight >}}

- The `.service.port` is taken from the `service.json` file.
- The `extra_config` in the third line is inserted as a JSON object using the `marshal` function from the `service.json` as well.
- A `range` iterates the array found under `endpoint.json` and key `example_group`. The variables inside the range are relative to the `example_group` content.
- An `include` in the extra_config inserts the content as is.
- Also notice the little trick `{{if $idx}},{{end}}` inside the loop. When it is not in the first element `0`, it will add a comma to prevent breaking the JSON format.

Notice that there is a `{{ range }}`. If you wanted to use it inside a template, and not the base file, you would need to include it inside a sub-template with ``{{ template "template.tmp" .endpoint.example_group }}``.

To parse the configuration use:

{{< terminal title="Check and test (-t) the config">}}
FC_ENABLE=1 \
FC_SETTINGS="$PWD/config/settings" \
FC_PARTIALS="$PWD/config/partials" \
krakend check -t -d -c "$PWD/config/krakend.json"
{{< /terminal >}}
