---
lastmod: 2018-11-02
date: 2017-01-21
linktitle: Developer Tools
menu:
  documentation:
    parent: developer
title: Developer Tools
weight: 10
---
<span class="badge badge-warning">This document is a draft</span>

There are some resources that make your life easier when developing with KrakenD. These tools are ment to be used only in development and **never in production**

# Generate graphs from configuration
The [config2dot](https://github.com/devopsfaith/krakend-config2dot) is a tool to create graphs automatically after reading your configuration file `krakend.json`.

<img alt="Config2Dot example" src="https://github.com/devopsfaith/krakend-config2dot/blob/master/docs/config_1.png?raw=true" class="img-fluid">

# Debugging the activity
## krakend-memviz
Adds a [DOT](https://en.wikipedia.org/wiki/DOT_(graph_description_language)) file exporter of request/response snapshots to your proxy stack for debug and development purposes. Do not use this in production as it will kill your performance.


## krakend-spew
Dumps every request/response passing through your proxy stacks for debug and development purposes. Do not use this in production as it will kill your performance.
