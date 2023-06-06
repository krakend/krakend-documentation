---
lastmod: 2020-04-10
date: 2020-04-10
linktitle: Mutual TLS Authentication (mTLS)
title: Securing B2B communication with mTLS
description: The mutual TLS authentication is used in business-to-business (B2B) applications where clients must provide a certificate to connect to KrakenD
weight: 50
notoc: true
menu:
  community_current:
    parent: "060 Authentication & Authorization"
---

**Mutual TLS authentication** (mTLS) is an authentication mechanism used traditionally in business-to-business (B2B) applications where clients provide a certificate that allows to connect to the KrakenD server.

The certificates must be recognized by your system's Certification Authority (CA) or be added under the `ca_certs` list.

## Configuring mutual authentication
From the configuration file perspective, Mutual TLS Authentication is no more than flag at the root level of the configuration.

When mTLS is enabled, **all KrakenD endpoints** require clients to provide a known client-side X.509 authentication certificate. KrakenD relies on the system's CA to validate certificates.

To enable it you need to add `enable_mtls` to your `tls` configuration:

```json
{
    "version": 3,
    "tls": {
      "public_key": "/path/to/cert.pem",
      "private_key": "/path/to/key.pem",
      "enable_mtls": true,
       "ca_certs": [
            "./rootCA.pem"
        ]
    }
}
```

{{< schema data="tls.json" >}}


Connections not having a recognized certificate in KrakenD's system CA, will be rejected. For further documentation on TLS, see the [`TLS` documentation](/docs/service-settings/tls/)

## mTLS example
To use mTLS you need to generate the client and server certificates. The following script example creates the needed files to enable mTLS. Notice that in the `CN` of the certificates we are adding `localhost` as we want to connect to KrakenD from and to localhost.

```sh
# Private key for the certificate authority
openssl genrsa -des3 -out rootCA.protected.key 2048
openssl rsa -in rootCA.protected.key -out rootCA.key
# Generate the CA
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem -subj "/C=US/ST=California/L=Mountain View/O=Your Organization/OU=Your Unit/CN=example.com"
# Generate a key for the client certificate
openssl genrsa -out client.key 2048
# Generate the certificate request for the client
openssl req -new -key client.key -out client.csr -subj "/C=US/ST=California/L=Mountain View/O=Your Organization/OU=Your Unit/CN=localhost"
# Sign the certificate request for the client
openssl x509 -req -in client.csr -extensions client -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out client.crt -days 500 -sha256

# Generate a key for the server certificate
openssl genrsa -out server.key 2048
# Generate the certificate request for the server
openssl req -new -key server.key -out server.csr -subj "/C=US/ST=California/L=Mountain View/O=Your Organization/OU=Your Unit/CN=localhost"
# Sign the certificate request for the server
openssl x509 -req -in server.csr -extensions server -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 500 -sha256
```

The KrakenD configuration needed is as follows (no endpoints used for this demo):

```json
{
    "version": 3,
    "$schema": "https://www.krakend.io/schema/v2.3/krakend.json",
    "port": 443,
    "tls": {
        "public_key": "./server.crt",
        "private_key": "./server.key",
        "enable_mtls": true,
        "ca_certs": [
            "./rootCA.pem"
        ],
        "disable_system_ca_pool": true
    }
}
```

At this moment KrakenD accepts only clients passing a valid certificate. Let's connect to the `/__health` endpoint:

{{< terminal title="Connect using mTLS" >}}
curl \
  --cacert rootCA.pem \
  --key client.key \
  --cert client.crt \
  https://localhost/__health
{"agents":{},"now":"2022-11-07 11:43:53.444657401 +0000 UTC m=+25.777003978","status":"ok"}
{{< /terminal >}}

If we don't provide the valid certs we get an error instead:

{{< terminal title="Connect without valid certs" >}}
curl -k https://localhost/__health
curl: (56) OpenSSL SSL_read: error:14094412:SSL routines:ssl3_read_bytes:sslv3 alert bad certificate, errno 0
{{< /terminal >}}
