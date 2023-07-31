---
lastmod: 2018-11-02
old_version: true
date: 2017-01-21
linktitle: Developer Tools
menu:
  community_v2.3:
    parent: "140 Developer Tools"
title: Custom developer tools
weight: 10
---

There are some additional resources that might help you when developing with KrakenD. These tools are meant to be used only in development and **never in production**, they are **not bundled with KrakenD** and are separate components that you must compile.

## Hot reload the configuration
There is an additional KrakenD Docker image using Reflex to watch the configuration directory and reload KrakenD when there are changes. This is very convenient while you are developing as it allows you to test new changes without having to restart manually and making the process less tedious.

You can use the Docker image `docker pull devopsfaith/krakend:watch`

[More information in our blog post](/blog/reloading-the-krakend-configuration/)

## Generating an image with the configuration
The [config2dot](https://github.com/krakend/krakend-config2dot) is a tool to create graphs automatically after reading your configuration file `krakend.json`. For instance:

![config2dot example](/images/documentation/config2dot.png)

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