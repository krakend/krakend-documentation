---
lastmod: 2023-03-30
date: 2019-02-22
linktitle: TLS
title: Enabling TLS for HTTPS and HTTP/2
description: The TLS settings define the parameters that KrakenD takes into account to handle incoming and outgoing HTTPS traffic.
weight: 30
notoc: false
menu:
  community_current:
    parent: "030 Service Settings"
skip_header_image: true
images:
- /images/documentation/diagrams/tls.mmd.png
---
The TLS settings define the parameters that the gateway takes into account to handle incoming and outgoing HTTPS traffic. We refer to this as:

- `tls`: **TLS settings**, or how the gateway handles incoming traffic as a server.
- `client_tls`: **Client TLS settings**, or how the gateway connects to your upstream services

![TLS diagram](/images/documentation/diagrams/tls.mmd.png)

{{< note title="Independent properties" type="tip" >}}
The properies `tls` and `client_tls` are independent of each other. You can declare one, both, or none.
{{< /note >}}




## TLS server settings
There are two different strategies when using TLS over KrakenD:

- Use TLS for HTTPS and HTTP/2 in KrakenD (this document)
- [Use a balancer with TLS termination in front of KrakenD](/docs/throttling/load-balancing/) (e.g., ELB, HAproxy)

If you want to listen with TLS, add a `tls` key at the service level (configuration's file root) with at least the public and private keys. When you add TLS, KrakenD listens **only using TLS**, and no traffic to plain HTTP is accepted.

If you want to enable mTLS see [Mutual TLS configuration](/docs/authorization/mutual-authentication/)

{{< note title="Secure by default" type="info" >}}
When you don't set any other parameters than stated below, KrakenD defaults to very strong security. **Only TLS 1.3 is accepted** (with its three cipher suites), and attempts to negotiate other versions will fail. Nevertheless, you can change this behavior.
{{< /note >}}

To start KrakenD with TLS, you need to provide a certificate for both the public and the private keys:

```json
{
  "version": 3,
  "tls": {
    "public_key": "/path/to/cert.pem",
    "private_key": "/path/to/key.pem"
  }
}
```

All TLS options for the server go inside the `tls` object:

{{< schema data="tls.json" >}}

## Client TLS settings
You can also set global TLS settings when KrakenD acts as a client, meaning that the gateway takes the role of the requesting user and fetches data with the upstream services.

All TLS options for the client go inside the `client_tls` object, and are similar to the server ones. When you set a `client_tls` in the configuration **the settings apply to all HTTP backends of all endpoints**.

If you need to **define TLS options for an individual backend** see [HTTP Client options](/docs/enterprise/backends/http-client/) {{< badge >}}Enterprise{{< /badge >}}

{{< schema data="client_tls.json" >}}

## Supporting older TLS 1.2 and below
Although we do not recommend downgrading your installation, this is the configuration you will need to support **older protocol versions**.

### Support old TLS v1.2
To support TLS v1.2 and 1.3 simultaneously, you need the following configuration:

```json
{
  "version": 3,
  "tls": {
    "public_key": "/path/to/cert.pem",
    "private_key": "/path/to/key.pem",
    "min_version": "TLS12"
  }
}
```

### Support archaic TLS versions
To support versions older than 1.2, specify the list of `cipher_suites` you want to enable (see below).

## Supported cipher suites
You can select which `cipher_suites` you'd like to support by passing an array of integers. The possible values of the cipher suites are listed below. The recommendation is to stay with TLS 1.3, or downgrade to TLS 1.2 when strongly needed. Below that should be out of the table.

**Default suites for TLS 1.3** are:

  - `4865`: TLS_AES_128_GCM_SHA256
  - `4866`: TLS_AES_256_GCM_SHA384
  - `4867`: TLS_CHACHA20_POLY1305_SHA256

**Default suites for TLS 1.2** are:

- `49199`: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- `49200`: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- `52392`: TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
- `49196`: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- `52393`: TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
- `49195`: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256

Other `cipher_suites` are supported but **not recommended**. Their existence is to support legacy systems:

  - `5`: TLS_RSA_WITH_RC4_128_SHA
  - `10`: TLS_RSA_WITH_3DES_EDE_CBC_SHA
  - `47`: TLS_RSA_WITH_AES_128_CBC_SHA
  - `53`: TLS_RSA_WITH_AES_256_CBC_SHA
  - `60`: TLS_RSA_WITH_AES_128_CBC_SHA256
  - `156`: TLS_RSA_WITH_AES_128_GCM_SHA256
  - `157`: TLS_RSA_WITH_AES_256_GCM_SHA384
  - `49159`: TLS_ECDHE_ECDSA_WITH_RC4_128_SHA
  - `49161`: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
  - `49162`: TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
  - `49169`: TLS_ECDHE_RSA_WITH_RC4_128_SHA
  - `49170`: TLS_ECDHE_RSA_WITH_3DES_EDE_CBC_SHA
  - `49171`: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
  - `49172`: TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
  - `49187`: TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
  - `49191`: TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256

## Generating certificates
You can acquire, use external tools, or self-generate your certificates.

For example, to generate a self-signed certificate from the command line, you can do the following:

{{< terminal title="Generate a certificate" >}}
openssl req -newkey rsa:2048 -new -nodes -x509 -days 365 -out cert.pem -keyout key.pem \
    -subj "/C=US/ST=California/L=Mountain View/O=Your Organization/OU=Your Unit/CN=localhost"
{{< /terminal >}}



## Common TLS errors

Self-signed certificates will show in the logs:

```log
http: TLS handshake error from 172.17.0.1:45474: local error: tls: bad record MAC
```
Old clients trying to connect to a newer TLS version, will produce errors as follows:
```log
http: TLS handshake error from 172.17.0.1:33698: tls: unsupported SSLv2 handshake received
http: TLS handshake error from 172.17.0.1:33710: tls: client offered only unsupported versions
```
Incompatible curves offered by client:
```
http: TLS handshake error from 172.17.0.1:33810: tls: no cipher suite supported by both client and server
http: TLS handshake error from 172.17.0.1:33814: tls: no ECDHE curve supported by both client and server
```

When the client offers a limited set of options that the server cannot accept. It will hang up the connection with an End Of File:

```
http: TLS handshake error from 172.17.0.1:33990: EOF
```

### SSL scan results for the default settings
The following output is the results of the [sslscan](https://github.com/rbsec/sslscan) command with a KrakenD configuration specifying the private and public keys in the configuration with no other additional `tls` settings:

```
Version: 2.0.15-7-gbc46606-static
OpenSSL 1.1.1u-dev  xx XXX xxxx

Connected to ::1

Testing SSL server localhost on port 443 using SNI name localhost

  SSL/TLS Protocols:
SSLv2     disabled
SSLv3     disabled
TLSv1.0   disabled
TLSv1.1   disabled
TLSv1.2   disabled
TLSv1.3   enabled

  TLS Fallback SCSV:
Server supports TLS Fallback SCSV

  TLS renegotiation:
Session renegotiation not supported

  TLS Compression:
Compression disabled

  Heartbleed:
TLSv1.3 not vulnerable to heartbleed

  Supported Server Cipher(s):
Preferred TLSv1.3  128 bits  TLS_AES_128_GCM_SHA256        Curve P-521 DHE 521
Accepted  TLSv1.3  256 bits  TLS_AES_256_GCM_SHA384        Curve P-521 DHE 521
Accepted  TLSv1.3  256 bits  TLS_CHACHA20_POLY1305_SHA256  Curve P-521 DHE 521

  Server Key Exchange Group(s):
TLSv1.3  128 bits  secp256r1 (NIST P-256)
TLSv1.3  192 bits  secp384r1 (NIST P-384)
TLSv1.3  260 bits  secp521r1 (NIST P-521)

  SSL Certificate:
Signature Algorithm: sha256WithRSAEncryption
RSA Key Strength:    2048

Subject:  localhost
Issuer:   localhost

Not valid before: Feb 28 11:44:05 2023 GMT
Not valid after:  Feb 28 11:44:05 2024 GMT
```