---
lastmod: 2024-03-11
old_version: true
old_version: true
date: 2016-07-01
linktitle: Upgrading the configuration
title: Upgrading KrakenD from an older version
description: Seamlessly migrate your KrakenD configuration to a new version. Hassle-free transition for your API Gateway
menu:
  community_v2.10:
    parent: "010 Configuration files"
weight: 1000
---
Upgrading to a new version of KrakenD is designed to be straightforward, thanks to our commitment to **maintaining backward compatibility** across versions within the same major release (e.g., within `2.x` versions). You can generally update KrakenD to a newer version without altering your configuration. However, to ensure optimal performance and access to the latest features, reviewing and adjusting your configuration is wise.

{{< note title="KrakenD's upgrade policy" type="info" >}}
Our policy focuses on **minimizing disruption by preserving compatibility with previous versions**. However, important changes can occur between versions, such as the deprecation of components, the introduction of superior alternatives, the relocation of properties, or the outright removal of outdated features. Although KrakenD aims to ensure your existing setup will continue to run, these changes may necessitate adjustments to your configuration for improved stability and performance.
{{< /note >}}

## Upgrade steps

1. **Review the [changelog](/changelog/)**. This document provides a chronological list of releases detailing new features, bug fixes, and possible breaking changes between major versions.
2. **Adjust your configuration and run the linter** as needed. Below, you'll find the changes between versions. Scroll down to your current version and apply all changes above it. Then, run [the linter](/docs/v2.11/v2.10/configuration/check/) (`krakend check --lint`), which is designed to scrutinize your configuration rigorously.
3. **Update the KrakenD binary**. Replace the existing binary file with the latest version. This process varies depending on how KrakenD was installed and whether it is container-based or not.
4. If you have custom plugins, you need to recompile them with the enterprise builder, and rename in your CI/CD the [test](/docs/v2.11/v2.10/extending/test-plugin/) and [check](/docs/v2.11/v2.10/extending/check-plugin/) plugin commands.

{{< note title="Jumping several versions" type="info" >}}
To upgrade when you are more than one version away from the latest, adjust the configuration for all the versions that the upgrade comprehends.
{{< /note >}}

_The list below is automatically generated based on the changelog._

{{% upgrade %}}

## Upgrade to v2.0 from v0.x or v1.x
The KrakenD 2.0 release is a major version that **simplifies the configuration** of `v1.x` and **standardizes field names** that were using different criteria to declare the attributes. The migration tool allows you to migrate from KrakenD `0.x` or `1.x` to `2.0`

{{< button-group >}}
{{< button url="https://github.com/krakend/krakend-config-migrator" text="Download migration tool" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z" /></svg>
{{< /button >}}
{{< /button-group >}}


### How to use the legacy migration tool

- Use `git` or similar DVCS to track the changes. Compare the differences at the end.
- Download the configuration [migration tool](https://github.com/krakend/krakend-config-migrator) and execute it passing the path to your KrakenD project
- Review the changes the migration tool did to your config and start the config with the new version

**If you have custom go plugins**, recompile them. KrakenD has now a command [`krakend check-plugin`](/docs/v2.11/v2.10/extending/check-plugin/) and [`krakend test-plugin`](/docs/v2.11/v2.10/extending/test-plugin/) to test them.

{{< note title="Special attention to short words" >}}
The migration script replaces words used by KrakenD in the past and are no longer supported that might collide with wording you use in your endpoints. Words like `whitelist` or `blacklist` will be replaced by `allow` and `deny`. Make sure to check the changes in the configuration and ensure that the migration tool didn't change any endpoint definition using those names.
{{< /note >}}

The migration tool will take care of what is described below for you, and is actually quite simple. For the most part, what it does is to rename configurations and namespaces. The following list is what it takes care of:

#### Renamed namespaces
The most visible change is that all non-core components (this is everything outside of [Lura](https://luraproject.org)) were declared inside an `extra_config` section, using a looong **namespace**. That namespace contained what could look like a URL (e.g., `github.com/devopsfaith/krakend-jose/validator`) and generated frequent misunderstandings year after year. Now, all namespaces have been categorized and simplified to a description of their functionality (e.g., `auth/validator`).

See the migration tool's source code for the complete list of renamed namespaces.

#### Consistent attribute naming
Another relevant change is that some attributes have been renamed to be consistent across all configurations. Prior to 2.0, some attributes used hyphenation (`hyphen-ation`), while others used snake case (`snake_case`) or camel case (`camelCase`). Now, we use `snake_case` everywhere if possible.

#### Removed deprecated elements
The final change is that all functionalities and attributes marked as deprecated in 1.4 have been removed.

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
