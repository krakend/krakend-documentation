---
lastmod: 2023-05-29
old_version: true
date: 2018-09-21
linktitle: Flexible Configuration
title: "Flexible Configuration: template-based config"
description: The Flexible Configuration allows you to declare the configuration using a templates system, multiple files, and variable injection.
source: https://github.com/krakend/krakend-flexibleconfig
since: 0.5
menu:
  community_v2.3:
    parent: "010 Configuration file(s)"
images:
- /images/krakend-flexible-config.png
weight: 30
---
The **Flexible Configuration** allows you to declare the configuration using **multiple files** and use a **templates system**, opening the door to multi-environment configurations and code reuse.

The Flexible Configuration enables a **template processor** based on [Go templates](https://golang.org/pkg/text/template/) and is enriched with [Sprig functions](http://masterminds.github.io/sprig/) and KrakenD functions.

You can encode your configuration files in any of the [supported formats](/docs/v2.3/configuration/supported-formats/) (`json`, `yaml`, `toml`, etc.), as the template is agnostic of its contents.

Use the Flexible Configuration when you need to:

- Split an extensive configuration into several files
- Inject variables, like environment settings, into the configuration
- Use placeholders and reusable code blocks to avoid repeating code
- Organize your code better when multiple developers are modifying the gateway
- Manage KrakenD using multiple repositories (and merging the files in the CI)
- Track changes, avoid conflicts, and review code more easily
- Have the full power of a templates system!

## How it works

The activation of the Flexible Configuration requires injecting the **environment variable** `FC_ENABLE=1` when running a krakend `run` or `check` command. Still, you can use **additional variables** depending on the features you'd like to enable.

### Flexible Configuration variables
The list of all recognized **environment variables** is:

- `FC_ENABLE=1`: Activates Flexible Configuration. You can use `1` or any other value (but `0` won't disable it!). The file passed with the `--config` flag is the **base template** and contains the references to any other templates.
- `FC_TEMPLATES=path/to/templates`: The path to the `templates` directory. These are evaluated using the Go templating system.
- `FC_SETTINGS=path/to/settings`: The path to the `settings` directory. Settings are JSON files that you can use to fill values in the templates, much similar to **env files** in other applications, but richer as you can use multiple files, structures, and nesting.
- `FC_PARTIALS=path/to/partials`: The path to the `partials` directory. Partial files are pieces of text that DON'T EVALUATE, and they are inserted in the placeholder "as is".
- `FC_OUT=file.json`: Saves the resulting configuration after rendering the template. It's required when you don't use JSON content (e.g., `FC_OUT=krakend.yml`), or when you need to pass the output to another program (like [`check --lint`](/docs/v2.3/configuration/check/)).


For instance, let's write a simple template `simple.tmpl` (go template emulating a json format):

```go-text-template
{
    "version": {{add 2 1}}
}
```

If the template system works, the server will start with a value `"version": 3`. We can test it with (Docker example):

{{< terminal title="Execute a simple template" >}}
docker run --rm -v "$PWD:/etc/krakend/" -e "FC_ENABLE=1" -e "FC_OUT=result.json" {{< product image >}} check -c simple.tmpl
Parsing configuration file: simple.tmpl
Syntax OK!
{{< /terminal >}}

We know that it works because KrakenD will fail with a different `version` value, but we can also check the contents of `result.json` for debugging purposes.

Let's now introduce the rest of the **optional** variables. For instance, let's assume you decided to organize your code as follows:

```
.
├── krakend.tmpl
└── config
    ├── partials
    │   └── file.txt
    ├── templates
    │   ├── telemetry.tmpl
    │   └── endpoints.tmpl
    └── settings
        ├── prod
        |   └── urls.json
        └── dev
            └── urls.json
```

Then you could run KrakenD from the terminal with this command:

{{< terminal title="Enabling flexible configuration with your custom dirs" >}}
FC_ENABLE=1 \
FC_SETTINGS="config/settings/prod" \
FC_PARTIALS="config/partials" \
FC_TEMPLATES="config/templates" \
FC_OUT="output.json" \
krakend run -c "krakend.tmpl"
{{< /terminal >}}

In the example above, notice that the `FC_SETTINGS` includes the path to the `prod`uction folder. This is how you would set a specific environment. You might inject here an env var if you have multiple environments. **The directory structure is completely up to you**.

{{< note title="Use a docker-compose" type="info">}}
Consider using a docker-compose in combination with the `:watch` image to speed up your development time.
{{< /note >}}

### Template syntax
The configuration file passed with the `-c` flag is treated as a **Go template** ([documentation](https://golang.org/pkg/text/template/)), and you can make use of all the power the template engine brings. In addition, the templating system is overloaded with [Sprig functions](http://masterminds.github.io/sprig/), and KrakenD functions, adding more features.

The data evaluations or control structures are easily recognized as they are surrounded by `{{` and `}}`. Any other text outside these delimiters is unprocessed text copied to the output as it is.

#### KrakenD-specific functions
To use the partials, templates, and settings as defined in the environment variables, you have the following functions available:

- `{{ marshal .var }}`: Insert a JSON structure taking the content from `.var`
- `{{ include "file.txt" }}`: Insert the content of the `file.txt` "as is". You can use any extension in these files.
- `{{ template "file.tmpl" . }}`: Renders the Go template `file.tmpl` passing all its variables as context. The context is the final `.` you can see in the block. The `file.tmpl` can access the context using `{{ . }}`. The context can be a simple value (like a string) or an object with nested elements.

See below for further explanation and examples.

#### Insert values from settings files
In the `FC_SETTINGS` directory, you can save multiple `.json` files with data structures inside that you can reuse in the templates. A `filename.json` is immediately available as the variable `{{ .filename }}` in the template.

For instance, if you have a file `settings/urls.json` with the following content:

```json
{
    "users_api": "https://users-api.mycompany.com",
    "inventory_api": "https://inventory-api.mycompany.com",
    "3rdparty": {
        "github": "https://api.github.com"
    }
}
```

You can access type in the template `{{ .urls.users_api }}`, and get `https://users-api.mycompany.com` when rendered. Or you could use `{{ .urls.3rdparty.github }}` and get `https://api.github.com`.

As you can see, the first word after the dot is the filename (without the extension), and the following dots traverse the objects to the final value.

#### Insert structures from settings files
When instead of a single value you need to insert a **JSON structure** (several elements), you need to use `marshal`.

```go-text-template
{{ marshal .urls }}
```

The example would write the entire content of the `urls.json` file.

#### Include an external file
To insert the content of an external partial file in-place use:

```go-text-template
{{ include "partial_file_name.txt" }}
```

**The content inside the partial template is not parsed**, and is inserted *as is* in plain text. The file is assumed to live inside the directory defined in `FC_PARTIALS` and can have any name and extension. Filenames referenced are **case sensitive**, and although your host operating system might work with case insensitive files (e.g., A docker volume on Mac), when copied to a Docker image not respecting the case will fail.

#### Include and process a sub-template
While the `include` is only meant to paste the content of a plain text file, the `template` gives you all the power of Go templating. The syntax is as follows:

```go-text-template
{{ template "template_name.tmpl" context }}
```

The template `template_name.tmpl` is executed and processed. The value of `context` is passed in the template as the context, meaning that the sub-template can access it using the dot `{{ . }}`. This context variable could be an object, such as `{{ template "environment.tmpl" .urls }}`, but it can also be another type, like a string: `{{ template "environment.tmpl" "production" }}`.

Go templates allow you to introduce handy stuff like conditionals or loops and allow you to create powerful configurations.

#### Sprig functions
Complementing the [Go built-in template language](http://golang.org/pkg/text/template/), and the KrakenD functions, the templates also adds more than 100 commonly used functions.

For instance, you could get a string from an environment variable `COMMIT_SHA`, and truncate it to 8 chars with:

```go-text-template
{
    "$schema": "https://www.krakend.io/schema/v2.3/krakend.json",
    "version": 3,
    "name": "Configuration commit {{ env "COMMIT_SHA" | trunc 8}}"
}
```

Sprig provides many functions in the following categories:

- String Functions: `trim`, `wrap`, `randAlpha`, `plural` and more.
    - String List Functions: `splitList`, `sortAlpha` and more.
- Integer Math Functions: `add`, `max`, `mul` and more.
    - Integer Slice Functions: `until`, `untilStep`
- Float Math Functions: `addf`, `maxf`, `mulf` and more.
- Date Functions: `now`, `date` and more.
- Defaults Functions: `default`, `empty`, `coalesce`, `fromJson`, `toJson`, `toPrettyJson`, `toRawJson`, `ternary`
- Encoding Functions: `b64enc`, `b64dec` and more.
- Lists and List Functions: `list`, `first`, `uniq` and more.
- Dictionaries and Dict Functions: `get`, `set`, `dict`, `hasKey`, `pluck`, `dig`, `deepCopy` and more.
- Type Conversion Functions: `atoi`, `int64`, `toString` and more.
- Path and Filepath Functions: `base`, `dir`, `ext`, `clean`, `isAbs`, `osBase`, `osDir`, `osExt`, `osClean`, `osIsAbs`
- Flow Control Functions: `fail`
- Advanced Functions
    - UUID Functions: `uuidv4`
    - OS Functions: `env`, `expandenv`
    - Version Comparison Functions: `semver`, `semverCompare`
    - Reflection: `typeOf`, `kindIs`, `typeIsLike` and more.
    - Cryptographic and Security Functions: `derivePassword`, `sha256sum`, `genPrivateKey` and more.
    - Network: `getHostByName`


{{< button-group >}}
{{< button url="http://masterminds.github.io/sprig/" text="Sprig documentation" >}}<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
  <path stroke-linecap="round" stroke-linejoin="round" d="M13.19 8.688a4.5 4.5 0 011.242 7.244l-4.5 4.5a4.5 4.5 0 01-6.364-6.364l1.757-1.757m13.35-.622l1.757-1.757a4.5 4.5 0 00-6.364-6.364l-4.5 4.5a4.5 4.5 0 001.242 7.244" />
</svg>
{{< /button >}}
{{< /button-group >}}


### Testing the configuration
As the configuration is now composed of several pieces, it's easy to make a mistake at some point. Test the syntax of all the files with the `krakend check` command and pay attention to the output to verify there aren't any errors.

You might also want to use the flag `FC_OUT` to write the content of the final file in a known path, so you can check its contents:

{{< terminal title="Checking the configuration" >}}
FC_ENABLE=1 \
FC_SETTINGS="$PWD/config/settings" \
FC_PARTIALS="$PWD/config/partials" \
FC_TEMPLATES="$PWD/config/templates" \
FC_OUT=out.json \
krakend check -t -d -c "$PWD/config/krakend.json"
{{< /terminal >}}

When there are errors, the output contains information to help you resolve it, e.g.:

```
ERROR parsing the configuration file: loading flexible-config settings:
- backends.json: invalid character '}' looking for beginning of object key string
```

## Tips & Tricks
The flexible configuration is a very simple tool (this documentation in fact is way larger than its implementation), but it can hold very complex setups. Here there are a few tips and tricks that you can use.

### Directory boilerplate
You can create the Flexible Configuration directory structure depicted above with the following:

{{< terminal title="Template boilerplate" >}}
mkdir -p config/{partials,settings,templates} config/settings/{prod,test}
{{< /terminal >}}

### Work faster with a docker-compose
Save the following `docker-compose.yml` and do a `docker-compose up`. This setup with the [`:watch` image](/docs/v2.3/developer/hot-reload/) will allow you to work locally with the latest version of KrakenD and apply the changes automatically every time you change a source file.

```yml
version: "3"
services:
  krakend:
    image: {{< product image >}}:watch
    volumes:
      - "./:/etc/krakend/"
    environment:
      - FC_ENABLE=1
      - FC_OUT=/etc/krakend/out.json
      - FC_PARTIALS=/etc/krakend/config/partials
      - FC_SETTINGS=/etc/krakend/config/settings/test
      - FC_TEMPLATES=/etc/krakend/config/templates
    command: ["run","-dc","krakend.tmpl"]
```

### Understanding the context
When you call a subtemplate from another template, you are in a `{{ range }}` (loop), or use a `{{ with }}`, you are using a specific context. The context is the variables you have available inside that block. When calling templates from templates, make sure to add the final dot `.` to pass all the settings files to the next template or pass those variables that are needed:

- `{{ template "hello.tmpl" . }}`: the hello template receives all setting files and works as its calling template.
- `{{ template "hello.tmpl" .urls.users_api }}`: receives only the string value of the users API.
- `{{ template "hello.tmpl" "hello world" }}`: receives only a constant string

### Using the $ notation
Similarly, when you are making a loop with a `range` or accessing a `with`, the variables inside are relative to its context one more time. But to access outsider variables you can use the `$` notation. For instance: `{{ $.urls.users_api }}`

### How to separate objects with commas
A lot of times, you need to iterate content and separate it using a comma. To do so, place the comma insertion at the beginning instead of the end when the index in the loop is not zero:

```go-text-template
{{ range $index, $endpoint := .endpoints_list }}
    {{if $index}},{{end}}
    {
        "endpoint": "{{ $endpoint.path }}",
        "backend": []
    }
{{end}}
```
### Remove white spaces and line breaks
When you use code `{{ blocks }}` on your templates, you can add a left dash `{{- blocks }}` to suppress preceding whitespaces and linebreaks or a right dash `{{ blocks -}}` to remove the following ones, or both `{{- blocks -}}`.

### Iterate all files under settings or all keys in a map
If for whatever reason you want to iterate all keys in a map, like the settings files, you have to be aware that you need the key names in the first place.

You need to use the `keys` function which returns all the key names in the map, and then you can access its contents using the function `index YourMapName yourKeyName`.

For instance, you can dump all settings files contents like this:

```go-text-template
{{ range $idx, $setting := keys .}}
    {{- if $idx}},{{end -}}
    "{{ $setting }}": {{marshal (index $ $setting)}}
{{end}}
```

In the example above the `range` iterates the key names of `.` (not the object itself), which is all the settings in the root template.

Then the `marshal` dumps all the contents provided. The dollar sign `$` inside the index represents all the content you have under `.` outside the range. As you are inside a range (a different scope) the `.` belongs to the range context, so you need to pass the dollar to access the outsider/parent context.

The `index` functions gives you access to an element of map, so `index $ $setting` is the equivalent in other languages to `$[$setting]`.

### Inserting an external file as base64
A few fields in KrakenD require you to set their value in base64 format instead of the raw counterpart. For example, sometimes you want to version control the raw file in an external file and reference it as base64. To do so, you could have a template `render_as_base64.tmpl` with the following content:

```go-text-template
{{/* Notice the dashes (-) at the beginning and end of the following code.
They remove all spaces and linebreaks that appear before and after when the result outputs. */}}
{{- $raw_content := include . -}}
{{- $raw_content | b64enc -}}
```
And call it in the `krakend.tmpl` like this:

```go-text-template
{
    "version": 3,
    "some_base_64_value": "{{template "render_as_base64.tmpl" "file.json"}}"
}
```

### Load dynamically all templates in a directory
Sometimes you want to "drag and drop" a new template into a folder and load it automatically in the configuration without referencing it. To do this, you must preprocess the template directory's contents into the file you will include.

Let's imagine your `krakend.tmpl` looks like this:

```go-text-template
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/v2.3/krakend.json",
    "endpoints": [
        {{ include "endpoints.tmpl" . }}
    ]
}
```
The `endpoints.tmpl` should contain the list of all the templates we want to include. One of the limitations of the Go `{{ include}}` is that you cannot use a variable to load a template, so we must pre-generated the contents.

There are many ways to generate this, and the following command is to save you time if you want to achieve this:

{{< terminal title="Dynamic file" >}}
tree -J -P "ep_*.tmpl" -I "endpoints.tmpl" \
| jq -r ' ( .[0].contents[].name | "{{ template \"\(.)\" . }}," )' \
| sed '$s/,$//' > endpoints.tmpl
{{< /terminal >}}

The command above will use `tree` to scan the contents of the template folder, using a theoretical pattern and generate the `endpoints.tmpl` file. The flags are:
- `-J` generates the tree output in JSON format.
- `-P` scans only for a particular pattern. In our example, templates beginning with `ep_`.
- `-I` excludes the file `endpoints.tmpl`
- The `jq` command prints in raw (`-r`) so quotes are not escaped and surrounds the template with the template syntax
- The final `sed` removes the last comma
- The output file `endpoints.tmpl` could look like this:

```bash
cat endpoints.tmpl
{{ template "ep_users.tmpl" . }}
{{ template "ep_checkout.tmpl" . }}
{{ template "ep_catalogue.tmpl" . }}
{{ template "ep_starred.tmpl" . }}
{{ template "ep_auth.tmpl" . }}
```

## Practical example
To demonstrate the usage of the flexible configuration, we will reorganize a configuration file into several pieces. This is a simple example to see the basics of the templates system:

```
.
└── config
    ├── krakend.tmpl
    ├── partials
    │   └── rate_limit_backend.tmpl
    └── settings
        ├── endpoint.json
        └── service.json
```

**partials/rate_limit_backend.tmpl**

In this file, we have written the content of the rate limit configuration for a backend. This file is inserted when included "as is":

```go
"qos/ratelimit/proxy": {
    "max_rate": "100",
    "capacity": "100"
}
```


**settings/service.json**

In the settings directory, we write all the files whose values can be accessed as variables.

```json
{
    "port": 8090,
    "default_hosts": [
        "https://catalog-api-01.srv",
        "https://catalog-api-02.srv",
        "https://catalog-api-03.srv"
    ],
    "extra_config": {
        "security/http": {
        "allowed_hosts": [],
        "ssl_proxy_headers": {
            "X-Forwarded-Proto": "https"
        },
        "ssl_certificate": "/opt/rsa.cert",
        "ssl_private_key": "/opt/rsa.key"
        }
    }
}
```

**settings/endpoint.json**

This file declares a couple of endpoints that feed on a single backend:

```json
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
```


**krakend.tmpl**

Finally, let's introduce the base template. It inserts the content of other files using `include`, uses the variables declared in the settings files, and writes json content with `marshal`.

Have a look at the highlighted lines:

{{< highlight go-text-template "hl_lines=3-5 7 9 12 15" >}}
    {
        "version": 3,
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
            ]}
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

To parse the configuration, use:

{{< terminal title="Check and test (-t) the config">}}
FC_ENABLE=1 \
FC_SETTINGS="$PWD/config/settings" \
FC_PARTIALS="$PWD/config/partials" \
krakend check -t -d -c "$PWD/config/krakend.tmpl"
{{< /terminal >}}
