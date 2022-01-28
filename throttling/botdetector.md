---
lastmod: 2020-07-10
date: 2019-09-15
linktitle: Bot detector
title: Control of bot traffic
weight: 30
source: https://github.com/devopsfaith/krakend-botdetector
images:
- /images/krakend-botdetector.png
menu:
  community_current:
    parent: "070 Traffic Management"
meta:
  since: 1.0
  source: https://security/bot-detector
  namespace:
  - security/bot-detector
  scope:
  - service
  log_prefix:
  - "[SERVICE: Gin][Botdetector]"
  - "[ENDPOINT: /foo][Botdetector]"
---

The **bot detector** module checks incoming connections to the gateway to determine if a bot made them, helping you detect and reject bots carrying out scraping, content theft, and form spam.

Bots are detected by inspecting the `User-Agent` and comparing its value with a set of configuration rules provided by you. The bot detector module **does not set any initial rules**, meaning that is up to you to decide the best rules for your use case, and choose how restrictive or permissive you are with bots.

As the bot detector module is flexible in its configuration, you can use it for other purposes than just discarding bots. For instance, you could set an allow rule for your mobile application `User-Agent` which would be allowed to interact with KrakenD and discard the rest of the traffic.

Discarded traffic receives a `403 Forbidden` status code.

## Configuring bot rules

The configuration rules of the bot detector have to be included inside the `extra_config`'s namespace `security/bot-detector` at the root level of your `krakend.json` file.

For instance:

    "extra_config": {
        "security/bot-detector": {
            "allow": ["MyAndroidClient/1.0", "Pingdom.com_bot_version_1.1"],
            "deny": ["a", "b"],
            "patterns": [
                "(Pingdom.com_bot_version_).*",
                "(facebookexternalhit)/.*"
            ],
            "cache_size": 10000
        }
    }

The available configuration options in the bot detector module are:

*   `allow`: An array with EXACT MATCHES of trusted user agents that can connect.
*   `deny`: An array of EXACT MATCHES of undesired bots, to reject immediately.
*   `patterns`: An array with all the **regular expressions** that define bots. Matching bots are rejected.
*   `cache_size`: Size of the LRU cache that helps speed the bot detection. The size is the mumber of users agents that you want to keep in memory.


Notice that the `allow` and the `deny` do not expect regular expressions, but **literal strings**. The purpose of this design is to get the best performance as comparing a literal string is much faster than evaluating a regular expression.

On the other hand, the `patterns` attribute expects regular expressions. The syntax is the same general syntax used by Perl, Python, and other languages. More precisely, it is the syntax accepted by [RE2](https://golang.org/s/re2syntax)

The order of evaluation of the rules is sequential in this order: `allow` -> `deny` -> `patterns`. When a user agent matches in any of the former evaluations, the execution ends, and the connection is accepted (allow) or rejected (deny and patterns).

### Building your bot rules

Fighting against spam, spiders, scrapping, theft, and bots is a problematic matter. There are different angles you can choose to combat it using the bot detection module.

Maybe you want to have a [massive list](https://github.com/ua-parser/uap-core/blob/master/regexes.yaml) of regular expressions of bots that are troubling you, and caching enabled.

Or perhaps you only require a single negative pattern that discards anything that you don't know is legit.

Whatever rules you decide to set in place, remember than allowing and denying are faster but are inflexible and require you to set the exact user-agent. On the other hand, regular expressions are very convenient, but the cost of evaluating them is higher in comparison.

### Caching

Evaluating every user agent against a **substantial list of patterns** can be a time-consuming operation. Even when we are talking about a few milliseconds, you can enable caching by setting `cache_size` and avoid reprocessing User-Agents checked before. Every millisecond counts!

The LRU caching system is in-memory and does not require running a separate set of servers, thus reducing the operation pain. There are neither cache expiration times, nor explicit cache evictions. When/if the cache is full, the least recently used (LRU) element is automatically replaced with the new one. An order of magnitude of megabytes should be enough to save the different User-Agent requests and combinations.

Set in the `cache_size` an integer with the fixed size of the cache (number of elements to store), or `0` to disable caching.
