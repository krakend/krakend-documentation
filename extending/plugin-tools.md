---
lastmod: 2019-01-15
date: 2019-01-14
notoc: true
linktitle: Plugin validation tools
title: Plugin and dependencies validator
weight: 20
menu:
  documentation:
    parent: extending
---

The plugin validator is an online tool that allows you to find problems with your plugin dependencies. Go plugins are strict on which versions of libraries you can use, so it's important to make sure that your dependencies are compatible with the selected KrakenD versions.

## Access the online plugin validation tools

The plugin validator checks your `go.sum` file to find problems and reports all associated problems. From which Go version is supported, to which individual libraries will conflict during runtime.

<a class="btn btn-secondary btn-lg" href="http://plugin-tools.krakend.io/validate">Go to plugin validator</a>

The dependencies list shows all libraries used by KrakenD. Make sure your plugins use the same versions:

<a class="btn btn-secondary btn-lg" href="http://plugin-tools.krakend.io">Go to dependencies finder</a>
