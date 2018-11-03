---
lastmod: 2018-09-29
date: 2018-09-21
linktitle: Flexible configuration
menu:
  documentation:
    parent: configuration
title: Flexible configuration
weight: 100
---
The **Flexible Configuration** component ([source code](https://github.com/devopsfaith/krakend-flexibleconfig)) is included in the KrakenD API Gateway and allows you to split the configuration file into several pieces for a more natural organization.

When the Flexible Configuration is enabled, KrakenD assumes that your configuration file is a template that needs compilation during start-up time. With this, you have the opportunity to produce a more sophisticated configuration file that utilizes variables and brings content from external files.

# When to use Flexible Configuration
A template system gives you full flexibility to work with the configuration file. It comes handy to:

- Split a large `krakend.json` file into several pieces
- Inject variables in the configuration
- Use placeholders and reusable code blocks
- Have the full power of the go template system!

# Requirements
The only requirement to use the Flexible Configuration is to encode the configuration file in `JSON` format as the package does not support other formats just yet.

# Usage
The activation of the package works via environment variables, as follows:

- `FC_ENABLE=1` to let KrakenD know that you are using Flexible Configuration. You can use `1` or any other value, just pass the variable ( `0` won't disable it!). The file passed with the `-c` flag is the base template.
- `FC_SETTINGS=dirname`: The path to the directory with all the settings files.
- `FC_PARTIALS=dirname`: The path to the directory with the partial templates included in the configuration file.
- `FC_OUT`: For debugging purposes, pass a filename where the final JSON result of parsing the template is saved.

For instance, let's assume you decided to organize your configuration as follows:

    .
    └── config
       ├── krakend.json
       ├── partials
       │   └── service_extra_config.tmpl
       └── settings
           └── db.json

Then you can run KrakenD from the terminal with this command:

    $ FC_ENABLE=1 \
    FC_SETTINGS="$PWD/config/settings" \
    FC_PARTIALS="$PWD/config/partials" \
    krakend run -c "$PWD/config/krakend.json"

## Template syntax
The configuration file passed with the `-c` flag is treated as a **Go template**, and you can make use of all the power the template engine brings. The data evaluations or control structures are easily recognized as they are surrounded by `{{` and `}}`. Any other text outside the delimiters is copied to the output unchanged.

### Include an external file
To insert the content of an external partial file in-place use:

    {{ include "partial_file_name.tmpl" }}

The file is assumed to live inside the directory defined in `FC_PARTIALS`.

### Insert values from settings files
In the `FC_SETTINGS` directory, you can save different `.json` files with data structures insides that you can reference in the templates.

For instance, if you have a file `settings/db.json` with the following content:

    {
        "host": "192.168.1.23",
        "port": 8766,
        "pass": "a-p4ssw0rd"
    }

You can access particular settings like this: `{{ .db.host }}`.

When instead of a single value you need to insert a whole JSON structure, you need to use the `marshall`.

    {{ marshal .db }}

## Testing the configuration
As the configuration is now composed of several pieces, it's easy to do a mistake in some point. Test the syntax of all the files is good with the `krakend check` command and pay attention to the output to verify there aren't any errors.

You might also want to use the flag `FC_OUT` to write the temporary file to a known path, so you can check its contents:

    FC_ENABLE=1 FC_SETTINGS="$PWD/config/settings" FC_PARTIALS="$PWD/config/partials" FC_OUT=out.json krakend check -d -c "$PWD/config/krakend.json"

When there are errors the output contains information to help you resolve it, e.g:

    ERROR parsing the configuration file: loading flexible-config settings:
    - backends.json: invalid character '}' looking for beginning of object key string

# Flexible configuration example
To demonstrate the usage of the flexible configuration we are going to reorganize a configuration file in several pieces. This is a simple example to see the basics of the templating system, and it does not intend to show a good way to organize and split the files:

    .
    └── config
        ├── krakend.json
        ├── partials
        │   └── rate_limit_backend.tmpl
        └── settings
            ├── endpoint.json
            └── service.json

**partials/rate_limit_backend.tmpl**

In this file we have written the content of the rate limit configuration for a backend. This file will be inserted when included "as is":

    "github.com/devopsfaith/krakend-ratelimit/juju/proxy": {
        "maxRate": "100",
        "capacity": "100"
    }

**settings/service.json**

In the settings directory we will write all the files that be offered you can write anything you want and the contents can be retrieved as variables.

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

This file declares a couple of endpoints that feed from a single backend:

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

Finally, the base template. It inserts the content of other files using `include`, uses the variables declared in the settings files and writes json content with `marshal`.

Have a look at the highlighted lines:

{{< highlight go "hl_lines=3-5 7 9 12 16 23" >}}
    {
        "version": 2,
        "port": "{{ .service.port }}",
        "extra_config": {{ marshal .service.extra_config }},
        "host": {{ marshal .service.default_hosts }},
        "endpoints": [
            {{ range .endpoint.example_group }}
            {
            "endpoint": "{{ .endpoint }}",
            "backend": [
                {
                    "url_pattern": "{{ .backend }}"
                }
            ]
            },
            {{ end }}
            {
            "endpoint": "/limited-endpoint",
            "backend": [
                {
                    "url_pattern": "/v1/slow-backend",
                    "extra_config": {
                        {{ include "rate_limit_backend.tmpl" }}
                    }
                }
                ]
            }
        ]
    }
{{< /highlight >}}

- The `.service.port` is taken from the `service.json` file.
- The `extra_config` in the third line is inserted as a JSON object using the `marshall` function from the `service.json` as well.
- A `range` iterates the array found under `endpoint.json` and key `example_group`. The variables inside the range are relative to the `example_group` content.
- Finally, an `include` at the bottom inserts the content as is.

To parse the configuration use:

    FC_ENABLE=1 \
    FC_SETTINGS="$PWD/config/settings" \
    FC_PARTIALS="$PWD/config/partials" \
    krakend check -d -c "$PWD/config/krakend.json"
