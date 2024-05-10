---
lastmod: 2023-10-23
old_version: true
date: 2020-04-10
linktitle: Mutual TLS Authentication (mTLS)
title: Mutual Authentication
description: Implement mutual authentication in KrakenD API Gateway to establish a secure and trusted communication channel between clients and APIs
weight: 50
notoc: false
menu:
  community_v2.6:
    parent: "080 Authentication & Authorization"
---

**mTLS** is an authentication mechanism used traditionally in business-to-business (B2B) applications where clients provide a certificate that allows to connect to the KrakenD server.

As KrakenD is a piece of software in the middle of two parts, there are different types of mTLS supported, that can work together or separately.

![mtls.mmd diagram](/images/documentation/diagrams/mtls.mmd.svg)


1. **Service mTLS**: When you require end-users to provide a certificate to connect to KrakenD.
2. **Client mTLS**: When you require KrakenD to provide a certificate to connect to your services.

In both cases, the certificates must be recognized by your system's Certification Authority (CA) or be added under the `ca_certs` list.

## Service mTLS Configuration (End-user to gateway)
From the configuration file perspective, Mutual TLS Authentication is no more than a flag `enable_mtls` under the `tls` section.

When mTLS is enabled, **all KrakenD endpoints** require clients to provide a known client-side X.509 authentication certificate. KrakenD relies on the system's CA to validate certificates.

To enable it you need a configuration like this:

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

And these are the options you can include under `tls`:
{{< schema version="v2.6" data="tls.json" >}}


**Important**: Connections not having a recognized certificate in KrakenD's system CA, will be rejected. For further documentation on TLS, see the [TLS documentation](/docs/v2.6/service-settings/tls/)

## Client mTLS Configuration (Gateway to service)
If you want that **all connections to backends** use mTLS, add the following configuration:

```json
{
    "version": 3,
    "client_tls": {
        "client_certs": [
            {
                "certificate": "cert.pem",
                "private_key": "cert.key"
            }
        ]
    }
}
```

{{< schema version="v2.6" data="client_tls.json" filter="client_certs" >}}

### Per-backend mTLS
If instead of enabling mTLS against all backends, you can enable mTLS in a specific backend only. This option is available only in the {{< badge color="denim" >}}Enterprise Edition{{< /badge >}}

An example configuration would be:

```json
{
  "endpoint": "/foo",
  "backend": [
    {
      "host": ["https://api-needing-mtls"],
      "url_pattern": "/foo",
      "extra_config": {
        "backend/http/client": {
          "client_tls": {
            "client_certs": [
              {
                "certificate": "cert.pem",
                "private_key": "cert.key"
              }
            ]
          }
        }
      }
    }
  ]
}
```
Configuration needed ({{< badge color="denim" >}}Enterprise Edition{{< /badge >}} only):

{{< schema version="v2.6" data="client_tls.json" filter="client_certs" >}}

This is the schema needed for client mTLS, but the [HTTP Client settings](/docs/enterprise/backends/http-client/) have many other options not related to mTLS.


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
    "$schema": "https://www.krakend.io/schema/v2.6/krakend.json",
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
