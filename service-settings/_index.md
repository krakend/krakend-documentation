---
lastmod: 2022-06-07
date: 2022-06-07
linktitle:  Service Settings
description: Changing the service settings of KrakenD API Gateway
title: Service Settings
weight: -1000
notoc: true
menu:
  community_current:
    parent: "030 Service Settings"
---
We call **service settings** (or the service layer) those parameters that allow you to change how KrakenD behaves globally (and not to a specific call). The service settings determine how you start the HTTP server, enforce security parameters, or define behavioral options like which reporting activities occur.

Examples of service settings are, the [listening port](/docs/service-settings/http-server-settings/), [disabling keep alives](/docs/service-settings/http-transport-settings/), enabling [metrics and traces](/docs/telemetry/), [listening https](/docs/service-settings/tls/), or enabling [CORS](/docs/service-settings/cors/) to name a few.

The service settings are written directly in the root of the configuration file or in its corresponding `extra_config`. For instance:

{{< highlight json "hl_lines=3-4">}}
{
    "version": 3,
    "port": 8080,
    "extra_config": {}
}
{{< /highlight >}}

Find the numerous options you have to change service settings in the rest of the documentation.