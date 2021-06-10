---
lastmod: 2020-04-10
date: 2020-04-10
linktitle: Mutual TLS Authentication (mTLS)
title: Securing B2B communication with mTLS
weight: 50
notoc: true
menu:
  community_v1.3:
    parent: "060 Authentication & Authorization"
---

Mutual TLS authentication (mTLS) is an authentication mechanism used traditionally in business-to-business (B2B) applications where clients provide a certificate that allows to connect to the KrakenD server.

## Configuring mutual authentication
From the configuration file perspective, Mutual TLS Authentication is no more than flag at the root level of the configuration.

When mTLS is enabled, **all KrakenD endpoints** require clients to provide a known client-side X.509 authentication certificate. KrakenD relies on the system's CA to validate certificates.

To enable it you need to add `enable_mtls` to your configuration:

    {
      "version": 2",
      "enable_mtls": true
    }

Connections not having a recognized certificate in KrakenD's system CA, will be rejected.