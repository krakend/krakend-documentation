---
date: 2023-06-08
lastmod: 2023-06-08
linktitle: Switch to Community or Enterprise
title: Switching to Community or Enterprise
description: What are the necessary steps to move from Community to Enterprise and vice-versa?
weight: 20
menu:
  community_current:
    parent: "999 Frequently Asked Questions"
---

If you are considering upgrading to KrakenD Enterprise or downgrading to KrakenD Community, these are the things you should have in mind.

## Downgrade to Community
If you are a KrakenD Enterprise Edition user and you want to switch to the Community Edition, these are the guidelines you should follow to keep your installation working.

- Obtain the [Community Edition](/download/) and replace it by the Enterprise one
- Make the configuration migration as described below
- Start the server with the modified (if necessary) configuration

### Configuration migration
You can run a configuration file designed for the KrakenD Enterprise in a Community Edition straight away, except if you have one of the following options that you should remove and will make your server panic during startup:

- [Dynamic routing](/docs/enterprise/endpoints/dynamic-routing/).
- [Endpoint with wildcards](/docs/enterprise/endpoints/wildcard/) (`/*`)

If you have any of the components above, remove them from the configuration.

The other [Enterprise-only features](/features/) are ignored and not loaded into the server, so whatever you leave or remove them from the configuration makes no difference to the server. Still, you should delete them to avoid confusion when reviewing the configuration later.

{{< note title="Enterprise features are lost" type="note" >}}
The Community Edition ignores unrecognized features from the Enterprise Edition and does not load any of its behavior. If you used critical components like [Security Policies](/docs/enterprise/security-policies/), you must know all the checks are gone even if they are in the configuration.
{{< /note >}}

## Upgrade to Enterprise
To upgrade a Community configuration to an Enterprise one, **no configuration changes are needed**. The only steps required are:

- Obtain [KrakenD Enterprise](/docs/enterprise/overview/installing/)
- Add the [`LICENSE` file](/docs/enterprise/overview/license-file/) to your project
- Start the new server

If you use the same version, everything you configured on the Community Edition works in the Enterprise Edition.