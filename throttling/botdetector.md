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

Bots are detected by inspecting the `User-Agent` header and checked against **your configuration rules**. You decide what is for you a bot and which ones you want to connect or not.


The configuration rules of the botdetector have to be included in the `extra_config` at the root level of your `krakend.json` file. For instance:

	"github_com/devopsfaith/krakend-botdetector": {
		"whitelist": ["c", "Pingdom.com_bot_version_1.1"],
		"blacklist": ["a", "b"],
		"patterns": [
			"(Pingdom.com_bot_version_).*",
			"(facebookexternalhit)/.*"
		],
		"cacheSize": 0
	}

The available options in the botdetector module are:

- `whitelist`: An array with EXACT MATCHES of all the bots you are trusting and willing to connect
- `blacklist`: An array of EXACT MATCHES of undesired bots
- `patterns`: An array with all the **regular expressions** that define bots. If one of these expressions match, the connection is rejected. You might want to have a look at [all these patterns](https://github.com/ua-parser/uap-core/blob/master/regexes.yaml).
- `cacheSize`: When the value is larger than 0, a LRU-based cache layer is added on top of the detector with the specified fixed size. This is used to avoid calculating the same User-Agent against all regexp strings.

The order of evaluation of rules is:

* whitelist -> If the whitelist is matched, no more checks are done and the connection is accepted.
* blacklist -> If the blacklist is matched, the connection is rejected
* patterns -> If blacklist or whitelist didn't match, all the patterns are tried in sequential order. Place the most susceptible matches first.

