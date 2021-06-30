---
lastmod: 2018-11-02
old_version: true
date: 2017-01-21
linktitle: Developer Tools
menu:
  community_v1.3:
    parent: "140 Developer Tools"
title: Developer Tools
weight: 10
---

There are some resources that make your life easier when developing with KrakenD. These tools are meant to be used only in development and **never in production**

## Hot reload the configuration
A Docker image using Reflex watches the configuration directory and reloads KrakenD when the configuration changes. This is very convenient while you are developing as it allows you to test new changes without having to restart manually and making the process less tedious.

[More information in our blog post](https://www.krakend.io/blog/reloading-the-krakend-configuration/)

## Generate graphs from configuration
The [config2dot](https://github.com/devopsfaith/krakend-config2dot) is a tool to create graphs automatically after reading your configuration file `krakend.json`.

![Config2Dot example](https://github.com/devopsfaith/krakend-config2dot/blob/master/docs/config_1.png?raw=true)

## Debugging the activity
### krakend-memviz
Adds a [DOT](https://en.wikipedia.org/wiki/DOT_(graph_description_language)) file exporter of request/response snapshots to your proxy stack for debug and development purposes. Do not use this in production as it will kill your performance.


### krakend-spew
Dumps every entity seen in the pipe: requests and responses passing through thew whole stack. Do not use this in production as it will kill your performance.

Dumps are stored in files like `<pipe>_<base64_endpoint/backend_name>_<timestamp>.txt`. E.g:

    $ ls
    2,0K 25 sep 19:12 backend_L3VzZXJzL3t7Lk5pY2t9fQ==_1537895547814979000.txt
    1,8K 25 sep 19:12 backend_LzIuMC91c2Vycy97ey5OaWNrfX0=_1537895547800941000.txt
    92K 25 sep 19:12 client_aHR0cHM6Ly9hcGkuYml0YnVja2V0Lm9yZy8yLjAvdXNlcnMva3BhY2hh_1537895547798571000.txt
    92K 25 sep 19:12 client_aHR0cHM6Ly9hcGkuYml0YnVja2V0Lm9yZy8yLjAvdXNlcnMva3BhY2hh_1537895547800824000.txt
    104K 25 sep 19:12 client_aHR0cHM6Ly9hcGkuZ2l0aHViLmNvbS91c2Vycy9rcGFjaGE=_1537895547814647000.txt
    1,9K 25 sep 19:12 client_basic_aHR0cHM6Ly9hcGkuYml0YnVja2V0Lm9yZy8yLjAvdXNlcnMva3BhY2hh_1537895547796264000.txt
    1,9K 25 sep 19:12 client_basic_aHR0cHM6Ly9hcGkuYml0YnVja2V0Lm9yZy8yLjAvdXNlcnMva3BhY2hh_1537895547798755000.txt
    2,7K 25 sep 19:12 client_basic_aHR0cHM6Ly9hcGkuZ2l0aHViLmNvbS91c2Vycy9rcGFjaGE=_1537895547812621000.txt
    2,3K 25 sep 19:12 proxy_L25pY2svOm5pY2s=_1537895547815244000.txt
    66K 25 sep 19:12 router_L25pY2sva3BhY2hh_1537895547816402000.txt
