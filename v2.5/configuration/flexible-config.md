---
lastmod: 2023-08-22
old_version: true
date: 2018-09-21
linktitle: Flexible Configuration
title: "Flexible Configuration: A template-based config for API Gateway"
description: Discover the flexible configuration options offered by KrakenD API Gateway, allowing you to customize and fine-tune your API gateway setup
source: https://github.com/krakend/krakend-flexibleconfig
since: 0.5
menu:
  community_v2.5:
    parent: "010 Configuration files"
images:
- /images/krakend-flexible-config.png
weight: 30
---
The **Flexible Configuration** allows you to declare the configuration using **multiple files** and use a **templates system**, opening the door to multi-environment configurations and code reuse.

The Flexible Configuration enables a **template processor** based on [Go templates](https://golang.org/pkg/text/template/) and is enriched with [Sprig functions](http://masterminds.github.io/sprig/) and KrakenD functions.

You can encode your configuration files in any of the [supported formats](/docs/v2.5/configuration/supported-formats/) (`json`, `yaml`, `toml`, etc.), as the template is agnostic of its contents.

Use the Flexible Configuration when you need to:

- Split an extensive configuration into several files
- Inject variables, like environment settings, into the configuration
- Use placeholders and reusable code blocks to avoid repeating code
- Organize your code better when multiple developers are modifying the gateway
- Manage KrakenD using multiple repositories (and merging the files in the CI)
- Track changes, avoid conflicts, and review code more easily
- Have the full power of a template system!

## How it works

Activating the Flexible Configuration requires injecting at least the **environment variable** `FC_ENABLE=1` when running a krakend `run` or `check` command and **additional variables** depending on the features you'd like to enable.

When you enable through the environment variables this feature, the configuration is loaded internally into a template parser that collects all the involved files and renders the final configuration file.

## Flexible Configuration variables

{{< note title="Different usage on Enterprise and Open Source" type="note" >}}
The **Enterprise version** of the Flexible Configuration uses the [Extended Flexible Config](/docs/enterprise/configuration/flexible-config/) engine, which **simplifies** the operation, allows using nested directories, recursivity, or the `$ref` operator, and needs none of the following variables amongst other features.

Still, all your open source-based templates are 100% compatible with the Enterprise counterpart.
{{< /note >}}


The **environment variables** of the flexible configuration are:

- `FC_ENABLE=1`: Activates Flexible Configuration. You can use `1` or any other value (but `0` won't turn it off!). The file passed with the `--config` flag is the **base template** and contains the references to any other templates.
- `FC_TEMPLATES=path/to/templates`: The path to the `templates` directory. These are evaluated using the Go templating system.
- `FC_SETTINGS=path/to/settings`: The path to the `settings` directory. Settings are JSON files that you can use to fill values in the templates, much similar to **env files** in other applications, but richer as you can use multiple files, structures, and nesting.
- `FC_PARTIALS=path/to/partials`: The path to the `partials` directory. Partial files are pieces of text that DON'T EVALUATE, and they are inserted in the placeholder "as is".
- `FC_OUT=file.json`: Saves the resulting configuration after rendering the template, useful for debugging, not required for runtime.


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

{{< note title="Use a docker compose" type="info">}}
Consider using a docker compose in combination with the `:watch` image to speed up your development time.
{{< /note >}}

## Template syntax
The configuration file passed with the `-c` flag is treated as a **Go template**, and you can use all the power the template engine brings. In addition, the templating system is overloaded with [Sprig functions](http://masterminds.github.io/sprig/), and KrakenD functions, adding more features.

The data evaluations or control structures are easily recognized as they are surrounded by `{{` and `}}`. Any other text outside these delimiters is unprocessed text copied to the output as it is.

Read the [Flexible Config Templates](/docs/v2.5/configuration/templates/) documentation to start playing with templates.

## Testing the configuration
We recommend using a Docker compose file to work faster with flexible configuration.

Save the following `docker-compose.yml` and do a `docker compose up`. This setup with the [`:watch` image](/docs/v2.5/developer/hot-reload/) will allow you to work locally with the latest version of KrakenD and apply the changes automatically whenever you change a source file.

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


As the flexible configuration is composed of several pieces, it's easy to make a mistake at some point. Test the syntax of all the files with the `krakend check` command and pay attention to the output to verify there aren't any errors. When there are errors, the output contains information to help you resolve it, e.g.:

```
ERROR parsing the configuration file: loading flexible-config settings:
- backends.json: invalid character '}' looking for beginning of object key string
```

The variable `FC_OUT` writes the content of the final file in a known path, so you can check its contents at any time.

If you don't use `docker compose`, you can also use flexible configuration as follows:

{{< terminal title="Checking the configuration" >}}
FC_ENABLE=1 \
FC_SETTINGS="$PWD/config/settings" \
FC_PARTIALS="$PWD/config/partials" \
FC_TEMPLATES="$PWD/config/templates" \
FC_OUT=out.json \
krakend check -t -d -c "$PWD/config/krakend.json"
{{< /terminal >}}