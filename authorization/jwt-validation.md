---
lastmod: 2018-11-03
date: 2018-11-03
linktitle: JWT Validation
title: JWT Validation
weight: 20
source: https://github.com/devopsfaith/krakend-jose
menu:
  documentation:
    parent: authorization
---
The JWT validation shields any amount of desired endpoints, forcing requests to the API gateway to provide a token **issued by a third party**. Verification of the token takes place in every request, including the check of the signature and optionally the assurance that its issuer, roles, and audience are sufficient to access the endpoint.

The **JOSE component** is responsible for validating the tokens.

A typical request requiring JWT validation includes in the `Authorization` header a bearer with the token:

    GET /resource HTTP/1.1
    Host: krakend.example.com
    Authorization: Bearer eyJhbGciOiJIUzI1NiIXVCJ9...TJVA95OrM7E20RMHrHDcEfxjoYZgeFONFh7HgQeyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IktyYWtlbkQiLCJpYXQiOjE1MTYyMzkwMjJ9.NVFYj2MhyvJjMESOg4ktIOfzak2ekD7IrCa9-UiO4QA

**And cookies?** Yes, you can also send the validation token in a **cookie**.

We consider a JWT token to be valid when is well formed, signed by a recognized issuer, unexpired, with some claims and is not marked as [revoked](/docs/authorization/revoking-tokens).

{{% note title="A note on JWT generation" %}}
When you generate tokens for end-users, make sure to set a **low expiration**. Tokens are supposed to have short lives and are recommended to expire in minutes or hours.
{{% /note %}}

# Basic JWT validation
The JWT validation is per endpoint and must be present inside every endpoint definition needing it. If several endpoints are going to require JWT validation consider using the [flexible configuration](/docs/configuration/flexible-config/) to avoid repetitive declarations.

Enable the JWT validation by adding the namespace `"github.com/devopsfaith/krakend-jose/validator"` inside the `extra_config` of the desired `endpoint`.

For instance, to protect the endpoint `/protected/resource`:

{{< highlight go "hl_lines=4-10" >}}
{
    "endpoint": "/protected/resource",
    "extra_config": {
        "github.com/devopsfaith/krakend-jose/validator": {
            "alg": "RS256",
            "audience": ["http://api.example.com"],
            "roles_key": "http://api.example.com/custom/roles",
            "roles": ["user", "admin"],
            "jwk-url": "https://albert-test.auth0.com/.well-known/jwks.json"
        }
    },
    "backend": [
        {
        "url_pattern": "/"
        }
    ]
}
{{< /highlight >}}

This configuration makes sure that:

- The token is well-formed and didn't expire
- The token has a valid signature
- The role of the user is either `user` or `admin` (taken from a key in the JWT payload named `http://api.example.com/custom/roles`)
- The token is not revoked in the bloomfilter (see [revoking tokens](/docs/authorization/revoking-tokens))

## JWT validation settings
The following settings are available for JWT validation. **Fields `alg` and `jwk-url` are mandatory** and the rest of the keys can be added or not at your best convenience.

Add them under the `"github.com/devopsfaith/krakend-jose/validator"` namespace:

- `alg`: *recognized string*. The hashing algorithm used by the issuer. Usually `RS256`.
- `jwk-url`: *string*. The URL to the JWK endpoint with the public keys used to verify the authenticity and integrity of the token.
- `cache`: *boolean*. Set this value to `true` to store the JWK public key in-memory for the next 15 minutes and avoid hammering the key server, recommended for performance. The cache can store up to 100 different public keys simultaneously.
- `audience`: *list*. Set when you want to reject tokens that do not contain an audience of the list.
- `roles_key`: When passing roles, the key name inside the JWT payload specifying the role of the user.
- `roles`: *list*. When set, the JWT token not having at least one of the listed roles are rejected.
- `issuer`: *string*. When set,  tokens not matching the issuer are rejected.
- `cookie_key`: *string*. Add the key name of the cookie containing the token when is not passed in the headers
- `disable_jwk_security`: *boolean*. When `true`, disables security of the JWK client and allows insecure connections (plain HTTP) to download the keys.
- `jwk_fingerprints`: *string list*. A list of fingerprints (the unique identifier of the certificate) for certificate pinning and avoid man in the middle attacks. Add fingerprints in base64 format.
- `cipher_suites`: *integers list*. Override the default cipher suites. Unless you have a legacy JWK, you don't need to add this value.

For the full list of recognized algorithms and cipher suites scroll down to the end of the document.

The following example contains every single option available:

{{< highlight go "hl_lines=3-24" >}}
"endpoint": "/foo"
"extra_config": {
    "github.com/devopsfaith/krakend-jose/validator": {
        "alg": "RS256",
        "jwk-url": "https://url/to/jwks.json",
        "cache": true,
        "audience": [
            "audience1"
        ],
        "roles_key": "department",
        "roles": [
            "sales",
            "development"
        ],
        "issuer": "http://my.api.com",
        "cookie_key": "TOKEN",
        "disable_jwk_security": true,
        "jwk_fingerprints": [
            "S3Jha2VuRCBpcyB0aGUgYmVzdCBnYXRld2F5LCBhbmQgeW91IGtub3cgaXQ=="
        ],
        "cipher_suites": [
            10, 47, 53
        ]
    }
}
{{< /highlight >}}

# A complete running example
The [KrakenD Playground](/docs/overview/playground/) demonstrates how to protect endpoints using JWT and includes an example ready to use using a [Single Page Application from Auth0](https://auth0.com/docs/applications/spa). To try it, [clone the playground](https://github.com/devopsfaith/krakend-playground) and follow the README.

# Supported hashing algorithms and cipher suites

## Hashing algorithms
Accepted values for the `alg` field are:

- `EdDSA`: EdDSA
- `HS256`: HS256 - HMAC using SHA-256
- `HS384`: HS384 - HMAC using SHA-384
- `HS512`: HS512 - HMAC using SHA-512
- `RS256`: RS256 - RSASSA-PKCS-v1.5 using SHA-256
- `RS384`: RS384 - RSASSA-PKCS-v1.5 using SHA-384
- `RS512`: RS512 - RSASSA-PKCS-v1.5 using SHA-512
- `ES256`: ES256 - ECDSA using P-256 and SHA-256
- `ES384`: ES384 - ECDSA using P-384 and SHA-384
- `ES512`: ES512 - ECDSA using P-521 and SHA-512
- `PS256`: PS256 - RSASSA-PSS using SHA256 and MGF1-SHA256
- `PS384`: PS384 - RSASSA-PSS using SHA384 and MGF1-SHA384
- `PS512`: PS512 - RSASSA-PSS using SHA512 and MGF1-SHA512

## Cipher suites
Accepted values for cipher suites are:

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

**Default suites** are:

- `49199`: TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- `49195`: TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- `49200`: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- `49196`: TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- `52392`: TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
- `52393`: TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305