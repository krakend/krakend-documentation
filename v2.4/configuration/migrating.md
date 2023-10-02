---
lastmod: 2018-10-28
old_version: true
date: 2016-07-01
linktitle: Migrating from 1.x or 0.x
menu:
  community_v2.4:
    parent: "010 Configuration file(s)"
title: Migrating config from KrakenD 1.x or 0.x
weight: 1000
---

The KrakenD 2.0 release is a major version that **simplifies the configuration** of `v1.x` and **standardizes field names** that were using different criteria to declare the attributes.

This migration allows you to:

- Migrate from KrakenD `0.x` to `2.x`
- Migrate from KrakenD `1.0` to `2.x`
- Migrate from KrakenD `1.2` to `2.x`
- Migrate from KrakenD `1.3` to `2.x`
- Migrate from KrakenD `1.4` to `2.x`

{{< button-group >}}
{{< button url="https://github.com/krakend/krakend-config-migrator" text="Download migration tool" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z" /></svg>
{{< /button >}}
{{< /button-group >}}


## Migrating to KrakenD 2.0 and above

- Use `git` or similar DVCS to track the changes. Compare the differences at the end.
- Download the configuration [migration tool](https://github.com/krakend/krakend-config-migrator) and execute it passing the path to your KrakenD project
- Review the changes the migration tool did to your config and start the config with the new version

**If you have custom go plugins**, recompile them. KrakenD has now a command [`krakend check-plugin`](/docs/v2.4/extending/check-plugin/) to test them.

{{< note title="Special attention to short words" >}}
The migration script replaces words used by KrakenD in the past and are no longer supported that might collide with wording you use in your endpoints. Words like `whitelist` or `blacklist` will be replaced by `allow` and `deny`. Make sure to check the changes in the configuration and ensure that the migration tool didn't change any endpoint definition using those names.
{{< /note >}}

## Changes from 1.x to 2.x
There is nothing else you need to do other than using the migration tool, but below we explain what are those changes.

The migration tool will take care of what is described below for you, and is actually quite simple. For the most part, what it does is to rename configurations and namespaces.

### Renamed namespaces
The most visible change is that all non-core components (this is everything outside of [Lura](https://luraproject.org)) were declared inside an `extra_config` section, using a looong **namespace**. That namespace contained what it could look like an URL (e.g: `github.com/devopsfaith/krakend-jose/validator`) and was generating frequent missunderstanding year after year. Now all nampespaces have been categorized and simplified to a description of its functionality (e.g.: `auth/validator`).

For the full list of renamed namespaces see the source code of the migration tool.

### Consistent attribute naming
Another relevant change is that some attributes have been renamed now to have a consistent naming accross all configurations. Prior to 2.0 some attributes name used hyphenation (`hyphen-ation`), while others used snake case (`snake_case`) or camel case (`camelCase`). Now we use `snake_case` everywhere if possible.

### Removed deprecated elements
The final change is that all functionalities and attributes that were marked as deprecated on 1.4 have been removed.

- `whitelist` is removed, and only `allow` is recognized now
- `blacklist` is removed, and only `deny` is recognized now
- `krakend-etc` is no longer included in the binary
- `krakend-consul`, the integration of consul for the JWT revoker, is no longer included in the binary.

Summing up, see the before and after of the following snippet which has 3 of the changes mentioned above.

**KrakenD 1**:

```json
{
    "endpoint": "/foo",
    "extra_config": {
        "github.com/devopsfaith/krakend-jose/validator" {
            "alg": "RS256",
            "jwk-url": "https://url/to/jwks.json"
        }
    },
    "backend": [
        {
            "url_pattern": "/foo",
            "whitelist": ["field1", "field2"]
        }
    ]
}
```


**KrakenD 2**:
Differences highlighted

{{< highlight json "hl_lines=4 6 12">}}
{
    "endpoint": "/foo",
    "extra_config": {
        "auth/validator": {
            "alg": "RS256",
            "jwk_url": "https://url/to/jwks.json"
        }
    },
    "backend": [
        {
            "url_pattern": "/foo",
            "allow": ["field1", "field2"]
        }
    ]
}
{{< /highlight >}}
