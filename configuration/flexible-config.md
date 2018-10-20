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
There might be times when you want to split the content of a large configuration file, use placeholders instead of hardcoding values, reuse variables or even whole blocks of code in the configuration file.

Instead of working with the final configuration file, the package **[krakend-flexibleconfig](https://github.com/devopsfaith/krakend-flexibleconfig)** allows you to use a template that will generate the final configuration file for you.

This is an example of a flexible configuration template:

{{< highlight go "hl_lines=3 10 19 24 32-33 39 44" >}}
{
    "version": 2,
    "port": {{ .Port }},
    "endpoints": [
        {
            "endpoint": "/combination/{id}",
            "backend": [
                {
                    "host": [
                        "{{ .Jsonplaceholder }}"
                    ],
                    "url_pattern": "/posts?userId={id}",
                    "is_collection": true,
                    "mapping": {
                        "collection": "posts"
                    },
                    "disable_host_sanitize": true,
                    "extra_config": {
                        "namespace1": {{ marshal .Namespace1 }}
                    }
                },
                {
                    "host": [
                        "{{ .Jsonplaceholder }}"
                    ],
                    "url_pattern": "/users/{id}",
                    "mapping": {
                        "email": "personal_email"
                    },
                    "disable_host_sanitize": true,
                    "extra_config": {
                        "namespace1": {{ marshal .Namespace1 }},
                        "namespace2": {{ marshal .Namespace2 }}
                    }
                }
            ],
            "extra_config": {
                "namespace3": { "foo": "bar" },
                "namespace2": {{ marshal .Namespace2 }}
            }
        }
    ],
    "extra_config": {
        "namespace2": {{ marshal .Namespace2 }}
    }
}
{{< /highlight >}}