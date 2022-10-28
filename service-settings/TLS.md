---
lastmod: 2021-06-13
date: 2019-02-22
linktitle: TLS
title: Enabling TLS for HTTPS and HTTP/2
weight: 30
notoc: true
menu:
  community_current:
    parent: "030 Service Settings"
---
There are two different strategies when using TLS over KrakenD:

- Use TLS for HTTPS and HTTP/2 in KrakenD (this document)
- Use a balancer with TLS termination in front of KrakenD (e.g., ELB, HAproxy)

In case you want to enable TLS in KrakenD you need to add a `tls` key at service level (configuration's file root) with at least the public key and the private key. When you add TLS, KrakenD listens **only using TLS**, and no traffic to plain HTTP is accepted.

## TLS Configuration
To start KrakenD with TLS you need to generate the certificate and provide both the public and the private key:

```json
{
  "version": 3,
  "tls": {
    "public_key": "/path/to/cert.pem",
    "private_key": "/path/to/key.pem"
  }
}
```

{{< schema data="tls.json" >}}

The list of cipher suites with its values is:

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
  - `49199`: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
  - `49195`: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
  - `49200`: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
  - `49196`: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
  - `52392`: TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
  - `52393`: TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305

**TLS 1.3**:

  - `4865`: TLS_AES_128_GCM_SHA256
  - `4866`: TLS_AES_256_GCM_SHA384
  - `4867`: TLS_CHACHA20_POLY1305_SHA256

**Default suites** are:

- `49199`: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- `49195`: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- `49200`: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- `49196`: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- `52392`: TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
- `52393`: TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305

## Generating certificates
You can acquire, use external tools, or self-generate your certificates.

For example, to generate a self-signed certificate from the command line you can do:

{{< terminal title="Generate a certificate" >}}
openssl req -newkey rsa:2048 -new -nodes -x509 -days 365 -out cert.pem -keyout key.pem -subj \"/C=US/ST=California/L=Mountain View/O=Your Organization/OU=Your Unit/CN=localhost\"
{{< /terminal >}}

When you are using **self-signed certificates**, you must add the certificates to the local CA, or at least add while developing the `allow_insecure_connections` setting to `true`. Example:

```json
{
  "version": 3,
  "allow_insecure_connections": true,
  "tls": {
    "public_key": "/path/to/cert.pem",
    "private_key": "/path/to/key.pem"
  }
}
```