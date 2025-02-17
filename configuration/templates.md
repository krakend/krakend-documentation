---
lastmod: 2023-08-28
date: 2018-09-21
linktitle: Configuration with templates
title: "Introduction to Templates"
description: Understanding how to work with Go templates in the flexible configuration and other components. Template syntax and practical snippets to resolve common situations.
menu:
  community_current:
    parent: "010 Configuration files"
images:
#- /images/documentation/diagrams/flexible-config.mmd.svg

weight: 31
---
There are **several components and features** in KrakenD that allow you to define configurations or content manipulations using **templates**.

Whether you are using templates with [flexible configuration](/docs/configuration/flexible-config/), a [Request generator](/docs/enterprise/backends/body-generator/) or [Response manipulation](/docs/enterprise/backends/response-body-generator/) the syntax you use is the same, and it's based on **Go templates** (as Helm, Kubernetes, and many other systems).

Our convention for saving templates, is using the `.tmpl` extension, although this is not enforced. This document provides a few direction to use templates.


## Template syntax
The templates use internally the Go `text/template` package and **Sprig functions**. For templates loaded in the Flexible Configuration, there are additional custom functions to load external resources.

There are two initial external documentation pages worth reading to get familiar with these, although you'll find practical information below:

- The [Go text/template documentation](https://golang.org/pkg/text/template/) sets the rules of templates.
- [Sprig functions](http://masterminds.github.io/sprig/)

### Basics of templates
You will recognize templates because their data evaluations or control structures use surrounding `{{` and `}}`. Any other text outside these delimiters is unprocessed text copied to the output as it is.

For instance, let's write a simple template `krakend.tmpl` (A Go template rendering in JSON format):

```go-text-template
{
    "version": {{add 2 1}}
}
```

The template above uses a Sprig function `add` that sums two numbers, and prints when executed:

```json
{
    "version": 3
}
```

**Templates are agnostic of the encoding** you want to express. The example above renders a JSON file, but you could write XML (e.g., when writing a SOAP request) or anything else.

A few basics to get started:

- Comments look like `{{/* a comment */}}` and can be multiline
- Variables set by KrakenD are under `{{ .variable_name }}`. Notice the starting dot `.`.
- Variables you assign can use the syntax `{{ $myvariable := "hello" }}`, and when a variable already exists with `{{ $myvariable = "hello2" }}`
- Conditionals use the syntax `{{ if CONDITION }}yes{{else}}no{{end}}`. You can also use `{{else if}}`. Empty values evaluate to `false`.
- Loops, or iterations use the syntax `{{ range .ELEMENT}}...{{end}}` or `{{ range .ELEMENT}}...{{else}}...{{end}}`. The `else` is used when the element you want to iterate is empty. Additionally you can use `{{break}}` and `{{continue}}` in loops.
- You can loop with assigned indexes and variables as `{{ range $key, $value := .ELEMENT}}...{{end}}`
- Access to elements using `{{with .ELEMENT}}...{{end}}`, or `{{with .ELEMENT}}yes{{else}}no{{end}}` when the variable is empty
- The context is represented with a starting dot `.` (see below)
- You can suppress preceding and following spaces from any block adding `-`, for instance: `{{- if true -}}...{{- end -}}`
- You can pipe [functions](https://pkg.go.dev/text/template#hdr-Functions) you can add to this

### Understanding the context (the dot)
When you execute a template you have an **initial data structure** available that contains different variables, like the settings files (in a flexible configuration), or the request data (in a request generator) to put a couple of different usages.

The template has access to the data using a period `.` and called "dot". You can print wherever you are the context like this:

```go-text-template
{{ toJson . }}
```

You can access the data structure traversing its keys using the dot operator. For instance, if you have a `.var` like `{"a": { "b": "hi" } }` you can access the value `hi` using `{{ .var.a.b }}`.

Variables can be passed as arguments to functions and actions. When you are in a `{{ range .var }}` (loop), or use a `{{ with .var}}`, you are passing a `.var` context. Inside the block you will have a **new dot**. For instance, considering `.var` has the structure above:

```go-text-template
{{ with .var.a }}
{{ .b }}
{{end}}
```
Prints `hi`. As you can see the context inside the with is different, and we don't access it like `.a.b`.

When calling templates from templates (flexible config), make sure to add the final dot `.` to pass all the settings files to the next template or pass those variables that are needed:

- `{{ template "hello.tmpl" . }}`: The hello template receives all setting files and works as its calling template.
- `{{ template "hello.tmpl" .urls.users_api }}`: receives only the string value of the user API.
- `{{ template "hello.tmpl" "hello world" }}`: receives only a constant string

Only in the {{< badge >}}Enterprise{{< /badge >}} edition you have an additional variable `.meta` holding directory metadata under the settings tree (maybe you want to traverse directory contents in a template).


### Using the `$` notation to access outsider context
As we have seen, when making a loop with a `range` or accessing a `with`, the variables inside are relative to its context. Still, you can access outsider variables with the `$` notation.

For instance with the same `{{.var}}` containing `{"a": { "b": "hi" } }`:

```go-text-template
{{ with .var.a }}
{{ .b }} and {{ $.var.a.b }}
{{end}}
```

Prints `hi and hi`, because the `$` allowed you to get content from outsider its context.

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

The `{{if $index}},{{end}}` is adding the comma except on the first item because the index `0` does not meet the condition.

### Remove white spaces and line breaks
When you use code `{{ blocks }}` on your templates, you can add a left dash `{{- blocks }}` to suppress preceding whitespaces and linebreaks or a right dash `{{ blocks -}}` to remove the following ones, or both `{{- blocks -}}`.

Suppressing additional spaces is irrelevant in JSON but not in YAML.

### Iterate all files under settings or all keys in a map
If you want to iterate all keys in a map, like the settings files, you must be aware that you need the key names in the first place.

You need to use the `keys` function, which returns all the key names in the map, and then you can access its contents using the function `index YourMapName yourKeyName`.

For instance, you can dump all settings file contents like this:

```go-text-template
{{ range $idx, $setting := keys .}}
    {{- if $idx}},{{end -}}
    "{{ $setting }}": {{marshal (index $ $setting)}}
{{end}}
```

In the example above, the `range` iterates the key names of `.` (not the object itself), which is all the settings in the root template.

Then, the `marshal` dumps all the contents provided. The dollar sign `$` inside the index represents all your content under `.` outside the range. As you are inside a range (a different scope) the `.` belongs to the range context, so you need to pass the dollar to access the outsider/parent context.

The `index` function gives you access to an element of the map, so `index $ $setting` is equivalent to `$[$setting]` in other languages.

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



### Sprig functions
Complementing the [Go built-in template language](http://golang.org/pkg/text/template/), Sprig adds more than 100 commonly used functions.

For instance, you could **inject secrets from environment variables**. Like:

```go-text-template
{
    "some_secret": "{{ env "SECRET_VARIABLE_NAME" }}"
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

### Flexible Configuration custom functions
The following custom functions are available for the **flexible configuration**, but **not in other components**.

To load external resources for the templates during runtime (partials, templates, and settings), you reference them in the templates as follows:

- `{{ include "file.txt" }}`: Inserts the content of the `file.txt` "as is". You can use any extension in these files.
- `{{ template "file.tmpl" . }}`: Renders the Go template `file.tmpl` passing all its variables as context. The context is the final `.` you can see in the block. The `file.tmpl` can access this context using `{{ . }}`. The context can be a simple value (like a string) or an object/map with nested elements.
- `{{ .setting_name }}`: All the settings files resolve to a tree that you can access in the templates. For instance, a `filename.json` is immediately available as the variable `{{ .filename }}` in the template.

For instance, having a file `settings/urls.json` with the following content:

```json
{
    "users_api": "https://users-api.mycompany.com",
    "inventory_api": "https://inventory-api.mycompany.com",
    "3rdparty": {
        "github": "https://api.github.com"
    }
}
```

You can refer to those values in the template like `{{ .urls.users_api }}` (which resolves to `https://users-api.mycompany.com`). Or you could use nested content like `{{ .urls.3rdparty.github }}` and get `https://api.github.com`.

As you can see, the first word after the dot is the filename (without the extension), and the following dots traverse the objects to the final value.

#### Insert structures from settings files
When instead of a single value, you need to insert a **JSON structure** (several elements), you need to use `marshal`.

```go-text-template
{{ marshal .urls }}
```

You can dump the context anywhere like this (useful for debugging):

```go-text-template
{{ . | marshal }}
```

The example would write the entire content of the `urls.json` file.

#### Include a partial
To insert the content of an external partial file in place use:

```go-text-template
{{ include "dir1/dir2/partial_file_name.txt" }}
```

**The content inside the partial template is not parsed** and is inserted *as is* in plain text. The file is assumed to live inside the directory defined as **partials** and can have any name and extension. Filenames referenced are **case sensitive**, and although your host operating system might work with case insensitive files (e.g., A docker volume on Mac) when copied to a Docker image not respecting the case will fail.

#### Include and process a sub-template
While the `include` is only meant to paste the content of a plain text file, the `template` gives you all the power of Go templating. The syntax is as follows:

```go-text-template
{{ template "template_name.tmpl" .some_context }}
```

The template `template_name.tmpl` is executed and processed. The depicted variable `.some_context` is passed in the template as the context.

#### The context
The context is data you pass to a template as if it were a single parameter of a function.

When the base template loads (e.g., the `krakend.tmpl`), it automatically **receives in the context the whole settings tree**. It means that if you dump the content of the context, you will see the entire tree made of all files and data structures in the settings dir.

If you want to pass **all the settings tree** to the calling template, write just a dot `.`, which stands for "everything". For instance:

```go-text-template
{{ template "template_name.tmpl" . }}
```

The called template can see the object you passed as the context under `{{ . }}`. In this case, all the settings are available as in the base template. But sometimes templates need a smaller object, a constant string, or a number as a parameter. You can also do this, for instance:

```go-text-template
{{ template "subtemplate1.tmpl" .some_setting.some_value }}
{{ template "ratelimit_per_minute.tmpl" 100 }}
```

Go templates allow you to introduce handy stuff like conditionals or loops and create powerful configurations.

#### Example file for flexible config
There is a lot of theory so far. To demonstrate the usage of the flexible configuration, we will reorganize a configuration file into several pieces. This is a simple example to see the basics of the templates system:

```
.
├── krakend.tmpl
├── partials
│   └── all_backends_extra_config.json
└── settings
    ├── endpoint.json
    └── service.json
```

**partials/all_backends_extra_config.json**

In this file, we have written the content of the rate limit configuration and circuit breaker we want for any backend. This file is inserted when included "as is", because it is a partial:

```json
{
    "$schema": "https://www.krakend.io/schema/v{{< product minor_version >}}/backend_extra_config.json",
    "qos/ratelimit/proxy": {
        "max_rate": 100,
        "capacity": 100
    },
    "qos/circuit-breaker": {
        "interval": 10,
        "max_errors": 5,
        "timeout": 5
    }
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
                    "extra_config": {{ include "all_backends_extra_config.json" }}
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

Notice that there is a `{{ range }}`. If you wanted to use it inside a template and not the base file, you would need to include it inside a sub-template with ``{{ template "template.tmp" .endpoint.example_group }}``.
