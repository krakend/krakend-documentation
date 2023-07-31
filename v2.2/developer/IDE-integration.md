---
lastmod: 2018-11-02
old_version: true
date: 2017-01-21
linktitle: IDE integration
title: IDE integration
weight: 20
notoc: true
images:
  - /images/documentation/krakend-ide-integration.png
skip_header_image: true
menu:
  community_v2.2:
    parent: "140 Developer Tools"
meta:
  since: 2.0
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
    "$schema": "https://www.krakend.io/schema/v2.2/krakend.json"
}
```

There is nothing else you need to do!