---
lastmod: 2018-09-21
date: 2018-09-21
linktitle: Flexible configuration
menu:
  documentation:
    parent: configuration
title: Flexible configuration
weight: 100
---
The **Flexible Configuration** component ([source code](https://github.com/devopsfaith/krakend-flexibleconfig)) is included in the KrakenD API Gateway and allows you to split the configuration file into several pieces for a more natural organization.

When the Flexible Configuration is enabled, KrakenD assumes that your configuration file is a template that needs compilation during start-up time. With this, you have the opportunity to produce a more sophisticated configuration file that utilizes a template system.

# Use case
A template system gives you full flexibility to work with the configuration file. It comes handy to:

- Split a large `krakend.json` file into several pieces
- Inject variables in the configuration
- Use placeholders and reusable code blocks
- Have the full power of the go template system!

# Requirements
The only requirement to use the Flexible Configuration is to encode the configuration file in `JSON` format as the package does not support other formats just yet.

# Usage
The activation of the package works via environment variables, as follows:

- `FC_ENABLE=1` to let KrakenD know that you are using Flexible Configuration. The file passed with the `-c` flag is the base template.
- `FC_SETTINGS=dirname`: The path to the directory with all the settings
- `FC_PARTIALS=dirname`: The path to the directory with the partial templates included in the configuration file

For instance, let's assume you decided to organize your configuration as follows:

    .
    └── config
       ├── krakend.json
       ├── partials
       │   └── service_extra_config.tmpl
       └── settings
           └── db.json

Then you can run KrakenD with this command:

    $ FC_ENABLE=1 \
    FC_SETTINGS="$PWD/config/settings" \
    FC_PARTIALS="$PWD/config/partials" \
    krakend run -c "$PWD/config/krakend.json"

## Template syntax
The configuration file is treated as a **Go template**, and you can make use of all the power the template engine brings. The data evaluations or control structures are easily recognized as they are surrounded by `{{` and `}}`. Any other text outside the delimiters is copied to the output unchanged.

### Include an external file
To insert the content of an external partial file in-place use:

    {{ include "partial_file_name.tmpl" }}

The file is assumed to live inside the directory defined in `FC_PARTIALS`.

### Insert values from settings files
In the `FC_SETTINGS` directory you can save different `.json` files with data structures insides that you can reference in the templates. 

For instance, if you have a file `settings/db.json` with the following content:

    {
        "host": "192.168.1.23",
        "port": 8766,
        "pass": "a-p4ssw0rd"
    }

You can access particular settings like this: `{{ .Db.Host }}`. Notice the variable uses the filename and the keys inside the structure with the first letter in upper case.

When instead of a single value you need to insert a whole JSON structure, you need to use the `marshall`.

    {{ marshal .Db }}

 
# Flexible configuration example
Your configuration templates can `include` other partial templates, insert settings from an external `JSON` file, or `marshal` the JSON value of a variable on the fly.

This is an example of a flexible configuration template that uses the following external files: `settings/service.json`, `settings/backend.json`, `extra.json`, and `partials/service_extra_config.tmpl`:

{{< highlight go "hl_lines=3 10 19 24 32-33 39 44" >}}
{
    "version": 2,
    "port": {{ .Service.Port }},
    "endpoints": [
        {
            "endpoint": "/combination/{id}",
            "backend": [
                {
                    "host": [
                        "{{ .Backend.Host }}"
                    ],
                    "url_pattern": "/posts?userId={id}",
                    "is_collection": true,
                    "mapping": {
                        "collection": "posts"
                    },
                    "disable_host_sanitize": true,
                    "extra_config": {
                        "namespace1": {{ marshal .Extra.Namespace1 }}
                    }
                },
                {
                    "host": [
                        "{{ .Backend.Host }}"
                    ],
                    "url_pattern": "/users/{id}",
                    "mapping": {
                        "email": "personal_email"
                    },
                    "disable_host_sanitize": true,
                    "extra_config": {
                        "namespace1": {{ marshal .Extra.Namespace1 }},
                        "namespace2": {{ marshal .Extra.Namespace2 }}
                    }
                }
            ],
            "extra_config": {
                "namespace3": { "foo": "bar" },
                "namespace2": {{ marshal .Extra.Namespace2 }}
            }
        }
    ],
    "extra_config": {{ include "service_extra_config.tmpl" }}
    }
}
{{< /highlight >}}