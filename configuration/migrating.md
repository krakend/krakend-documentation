---
lastmod: 2018-10-28
date: 2016-07-01
linktitle: Migrating from 1.x or 0.x
menu:
  community_current:
    parent: "010 Configuration file(s)"
title: Migrating config from KrakenD 1.x or 0.x
weight: 100
---

The KrakenD 2.0 release is a major version that **simplifies the configuration** of `v1.x` and **standardizes field names** that were using different criteria to declare the attributes.

This migration allows you to:

- Migrate from KrakenD `0.x` to `2.x`
- Migrate from KrakenD `1.0` to `2.x`
- Migrate from KrakenD `1.2` to `2.x`
- Migrate from KrakenD `1.3` to `2.x`
- Migrate from KrakenD `1.4` to `2.x`

{{< button-group >}}
{{< button url="https://github.com/devopsfaith/krakend-config-migrator" text="Download migration tool" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z" /></svg>
{{< /button >}}
{{< /button-group >}}


## Short read - Migrating to KrakenD 2.0 and above

- Use `git` or similar DVCS to track the changes. Compare the differences at the end.
- Open your configuration file(s), such as `krakend.json`, and replace `"version": 2` with `"version": 3`. **This is the only change by hand.**
- Download the migration binary for your architecture (button above) and execute it passing the path to the folder where your configuration is (absolute or relative).
- If you use open-source with Docker, replace `devopsfaith/krakend` with `krakend` or `krakend:2`, or `krakend:2.0` (now it's an official Docker image!)
- Review the changes the migration tool did to your config, and delete the migrator.

And only **if you are compiling custom go plugins**, recompile them using the new libraries versions (guide to [write plugins](/docs/extending/writing-plugins/))

{{< note title="Special attention to short words" >}}
The migration script replaces words used by KrakenD in the past that might collide with wording you use in your endpoints. Words like `whitelist` or `blacklist` will be replaced. Make sure to check changes in the configuration.
{{< /note >}}


## Long read - Changes and differences explained
The migration tool will take care of what is described below for you, but it's not magic. What it does is actually quite simple, and for the most part, what it does is to rename configurations and namespaces.

### Renamed namespaces
The first relevant change is that all non-core components (this is everything outside of [Lura](https://luraproject.org)) were declared inside an `extra_config` section, using a looong key name (**namespace**). That namespace contained what it could look like an URL (e.g: `github.com/devopsfaith/krakend-jose/validator`) and was generating frequent missunderstanding. Now they have been categorized and simplified to a description of its functionality (e.g.: `auth/validator`).

For the full list of renamed namespaces see the source code of the migration tool.

### Consistent attribute naming
Another relevant change is that some attributes have been renamed now to have a consistent naming accross all configurations. Prior to 2.0 some attributes name used hyphenation (`hyphen-ation`), while others used snake case (`snake_case`) or camel case (`camelCase`). Now we are using `snake_case` everywhere whenever possible.

### Removed deprecated elements
The final change is that all functionalities and attributes that were marked as deprecated on 1.4 have been removed.

- `whitelist` is removed, and only `allow` is recognized now
- `blacklist` is removed, and only `deny` is recognized now
- `krakend-etc` is no longer included in the binary
- `krakend-consul` is no longer included in the binary (integration of consul for the Bloomfilter JWT revoker)

Summing up, see the before and after of the following snippet which has 3 of the changes mentioned above.

**KrakenD 1**:

{{< highlight json >}}
{
    "endpoint": "/foo",
    "extra_config": {
        "auth/validator" {
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
