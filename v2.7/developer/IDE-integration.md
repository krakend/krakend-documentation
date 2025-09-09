---
lastmod: 2018-11-02
old_version: true
date: 2017-01-21
linktitle: IDE integration
title: IDE Integration Guide for Developers
description: Enhance your development workflow by integrating KrakenD with your favorite IDE. Follow our comprehensive guide to seamlessly integrate KrakenD into your development environment.
weight: 300
notoc: true
images:
  - /images/documentation/krakend-ide-integration.png
skip_header_image: true
menu:
  community_v2.7:
    parent: "170 API Documentation and Dev Tools"
meta:
  since: v2.0
  source: https://github.com/krakend/krakend-schema

---
Automatic validation as you type, showing documentation while hovering an attribute, explanation of errors, and autocompletion of properties, are features that you get automatically while working with KrakenD.

For users of **Visual Studio Code**, **Android Studio**, **JetBrains** editors (PHPStorm, PyCharm, GoLand, WebStorm, IntelliJ IDEA...), **Eclipse**, and other IDEs that have built-in json schema validation capabilities, there is nothing to install to have these features. Other editors can be used as well, but you will likely need to instal a JSON schema validator.

This is how it could look like:

![Visual Code integration](/images/documentation/krakend-ide-integration.png)

## Editor integration for KrakenD files
KrakenD has published an updated JSON-schema definition ([source](https://github.com/krakend/krakend-schema)) to validate configuration files from your IDE automatically. The editors with built-in json-schema validation will offer this feature **without installing any additional plugin**. All you need to do, is add in the beginning of your `krakend.json` configuration file a line specifing the schema:


```json
{
    "$schema": "https://www.krakend.io/schema/v2.7/krakend.json"
}
```

If you want to point to the latest version of KrakenD, and not to a specific version, you can add:


```json
{
    "$schema": "https://www.krakend.io/schema/krakend.json"
}
```


There is nothing else you need to do!

Part of the URL is the KrakenD version you want to validate, notice that it does not contain the patch number (e.g.: `vA.B` instead of `vA.B.C`).

## Highlighting on Flexible Configuration templates
If you are working with flexible configuration, look in your favourite's editor marketplace extensions to deal with go `text/template` files (not html/template) that support code highlighting.

Some editors also allow you the combination of templating + JSON format, so you can work with both.