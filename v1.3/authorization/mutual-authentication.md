---
lastmod: 2020-04-10
old_version: true
date: 2020-04-10
linktitle: Mutual TLS Authentication (mTLS)
title: Securing B2B communication with mTLS
weight: 50
notoc: true
menu:
  community_v1.3:
    parent: "060 Authentication & Authorization"
---
**Mutual TLS authentication** (mTLS) is an authentication mechanism used traditionally in business-to-business (B2B) applications where clients provide a certificate that allows to connect to the KrakenD server.

The certificates must be recognized by your system's Certification Authority (CA). KrakenD relies on the machine where is running.

## Configuring mutual authentication
From the configuration file perspective, Mutual TLS Authentication is no more than flag at the root level of the configuration.

When mTLS is enabled, **all KrakenD endpoints** require clients to provide a known client-side X.509 authentication certificate. KrakenD relies on the system's CA to validate certificates.

To enable it you need to add `enable_mtls` to your `tls` configuration:

{{< highlight json >}}
{
    "version": 2,
    "tls": {
      "public_key": "/path/to/cert.pem",
      "private_key": "/path/to/key.pem",
      "enable_mtls": true
    }
}
{{< /highlight >}}

Connections not having a recognized certificate in KrakenD's system CA, will be rejected. For further documentation on TLS, see the [`TLS` documentation](/docs/service-settings/tls/)
