---
lastmod: 2019-09-15
date: 2019-09-15
notoc: true
linktitle: Logstash
title: Logstash
weight: 50
source: https://github.com/devopsfaith/krakend-logstash
aliases:
- /docs/logging-metrics-tracing/logstash/
menu:
  documentation:
    parent: logging
---
If you want to log using the Logstash standard via stdout, you have to add the `krakend-logstash` integration in the
root level of your `krakend.json`, inside the `extra_config` section. **The `gologging` needs to be enabled too**.

For instance:

    "extra_config": {
      "github_com/devopsfaith/krakend-logstash": {
        "enabled": true
      }
      "github_com/devopsfaith/krakend-gologging": {
          "level": "INFO",
          "prefix": "[KRAKEND]",
          "syslog": false,
          "stdout": true,
          "format": "logstash"
      }
    }
