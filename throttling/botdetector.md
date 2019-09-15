---
lastmod: 2019-09-15
date: 2019-09-15
linktitle: Bot detector
title: Control of bot traffic
weight: 80
since: 1.0
notoc: true
source: https://github.com/devopsfaith/krakend-botdetector
menu:
  documentation:
    parent: throttling
---
The **bot detector** module checks incoming connections to the gateway to determine if they were made by a bot or not. It helps you actively detect bots carrying out scraping, content theft, and form spam.

Bots are detected by inspecting the `User-Agent` header and checked against your configuration. The bot detector module can blacklist or whitelist bots according to your preferences.

The configuration of the botdetector has to be included in the `extra_config` at the root level of your `krakend.json` file.

	"github_com/devopsfaith/krakend-botdetector": {
		"blacklist": ["a", "b"],
		"whitelist": ["c", "Pingdom.com_bot_version_1.1"],
		"patterns": [
			"(Pingdom.com_bot_version_).*",
			"(facebookexternalhit)/.*"
		],
		"cacheSize": 0
	}

The available options in the botdetector module are:

- `blacklist`: An array with all the undesired bot
- `whitelist`: An array with all the bots you are willing to accept
- `patterns`: An array with all the regular expressions that define bots. You might want to have a look at [all these patterns](https://github.com/ua-parser/uap-core/blob/master/regexes.yaml).
- `cacheSize`: When the value is larger than 0, a LRU-based cache layer is added on top of the detector with the specified fixed size.