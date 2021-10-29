---
lastmod: 2018-10-28
date: 2016-07-01
linktitle: Migrating from 1.x or 0.x
menu:
  community_current:
    parent: "010 Configuration file(s)"
title: Migrating config from KrakenD 1.x or 0.x
notoc: true
weight: 100
---

The KrakenD 2.0 release is a major version that **simplifies the configuration** of `v1.x` and **standardizes field names** that were using different criteria to declare the attributes.

Migrating from older versions of KrakenD is a straightforward process if you use the [Config migration tool](https://github.com/devopsfaith/krakend-config-migrator). This migration allows you to:

- Migrate from KrakenD `0.x` to `2.x`
- Migrate from KrakenD `1.0` to `2.x`
- Migrate from KrakenD `1.2` to `2.x`
- Migrate from KrakenD `1.3` to `2.x`
- Migrate from KrakenD `1.4` to `2.x`

## Steps to migrate to KrakenD 2.0 and above

- Version your configuration. Use git or alike to track the changes before and after applying the migration tool.
- Open your configuration file(s) and replace `"version": 2` with `"version": 3`.
- Download the migration binary for your architecture and execute it using the path to the folder where your configuration is.
- If you use Docker, replace `devopsfaith/krakend` with `krakend` or `krakend:2`
- Review the changes the migration tool did to your config
- **New container is not plugin friendly**, use container `krakend:2-debian`

## Changes and differences between KrakenD 1 and KrakenD 2 explained
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

**KrakenD 2**:

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
