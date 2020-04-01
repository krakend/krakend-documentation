---
lastmod: 2020-03-28
date: 2018-11-03
linktitle: JWT Validation
title: JWT Validation
weight: 20
source: https://github.com/devopsfaith/krakend-jose
menu:
  documentation:
    parent: authorization
---
The JWT validation **protects endpoints from public usage**, forcing calls to the API gateway to provide a valid token to access its contents.

The generation of the token itself has to be **driven by a third party**, although the user calls can be proxied through KrakenD. If you don't have an identity server yet you still can [sign tokens through KrakenD](/docs/authorization/jwt-signing/) 

The internal component responsible for validating tokens is called **krakend-jose**.

## JWT tokens
KrakenD uses standard JWT tokens to protect endpoints, using JSON Web Signature (**JWS**), to check the tokens' digital signature ensuring the integrity of the contained claims and protecting against attacks using tampered tokens.

A JWT token is a `base64` encoded string with the structure **header.payload.signature**.

A typical request to an endpoint requiring JWT validation includes a `Bearer` in the `Authorization` header:
{{< terminal >}}
GET /resource HTTP/1.1
Host: krakend.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIXVCJ9.(truncated).ktIOfzak2ekD7IrCa9-UiO4QA
{{< /terminal >}}

Or optionally, you can send the token **inside a cookie** instead.

### JWT header requirements
When KrakenD decodes the `base64` token string passed in the `Bearer` or the cookie, it expects to find in its **header** section the following 3 fields:

	{
	  "alg": "RS256",
	  "typ": "JWT",
	  "kid": "MDNGMjU2M0U3RERFQUEwOUUzQUMwQ0NBN0Y1RUY0OEIxNTRDM0IxMw"
	}

The values of the `alg` and `kid` depend on your implementation, but they must be present.

The value provided in the `kid` string **must match the `kid` value of one of the keys exposed at the URL provided in the `jwk-url`** to verify the signature. The example above used [this public key](https://albert-test.auth0.com/.well-known/jwks.json), notice how the `kid` matches both the single key present in the JWK document and in the token header.

KrakenD is built with security in mind and uses JWS (instead of plain JWT or JWE), and the `kid` points to the right key in the JWS. This is why this entry is mandatory to validate your tokens. 

{{< note title="Important!" >}}
Make sure you are declaring the right `kid` in your JWT. Paste a token in a [debugger](https://jwt.io/#debugger-io) to find out.
{{< /note >}}

### Validation process
KrakenD does the following validation to let users hit protected endpoints:

- The `jwk-url` must be accessible by KrakenD at all times (caching is available)
- The token is [well formed](https://jwt.io/#debugger-io)
- The `kid` in the header is listed in the `jwk-url`.
- The content of the JWK Keys (`k`) is **base64** urlencoded
- The algorithm `alg` is supported by KrakenD and matches exactly the one used in the endpoint definition.
- The token hasn't expired
- The signature is valid.
- The given `issuer` matches (if present in the configuration)
- The given `audience` matches (if present in the configuration)
- The given claims are within the endpoint accepted `roles` (if present in the configuration))

The configuration allows you to define the set of required roles. A user who passes a token with roles `A` and `B`, can access an endpoint requiring `"roles": ["A","C"]` as it has one of the required options (`A`).

When you generate tokens for end-users, make sure to set a **low expiration**. Tokens are supposed to have short lives and are recommended to expire in a few minutes or hours.

## Basic JWT validation
The JWT validation is per endpoint and must be present inside every endpoint definition needing it. If several endpoints are going to require JWT validation consider using the [flexible configuration](/docs/configuration/flexible-config/) to avoid repetitive declarations.

Enable the JWT validation by adding the namespace `"github.com/devopsfaith/krakend-jose/validator"` inside the `extra_config` of the desired `endpoint`.

For instance, to protect the endpoint `/protected/resource`:

{{< highlight JSON "hl_lines=4-10" >}}
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
- The token is not revoked in the bloom filter (see [revoking tokens](/docs/authorization/revoking-tokens))

### JWT validation settings
The following settings are available for JWT validation. **Fields `alg` and `jwk-url` are mandatory** and the rest of the keys can be added or not at your best convenience.

Add them under the `"github.com/devopsfaith/krakend-jose/validator"` namespace:

- `alg`: *recognized string*. The hashing algorithm used by the issuer. See the [hashing algorithms](#hashing-algorithms) section for comprehensive list of supported algorithms.
- `jwk-url`: *string*. The URL to the JWK endpoint with the public keys used to verify the authenticity and integrity of the token.
- `cache`: *boolean*. Set this value to `true` to store the required keys (from the JWK descriptor) in memory for the next `cache_duration` period and avoid hammering the key server, recommended for performance. The cache can store up to 100 different public keys simultaneously.
- `cache_duration`: *int*. Change the default duration of 15 minutes. Value in **seconds**.
- `audience`: *list*. Set when you want to reject tokens that do not contain the given audience.
- `roles_key`: When passing roles, the key name inside the JWT payload specifying the role of the user.
- `roles`: *list*. When set, the JWT token not having at least one of the listed roles are rejected.
- `issuer`: *string*. When set,  tokens not matching the issuer are rejected.
- `cookie_key`: *string*. Add the key name of the cookie containing the token when is not passed in the headers
- `disable_jwk_security`: *boolean*. When `true`, disables security of the JWK client and allows insecure connections (plain HTTP) to download the keys. Useful for development environments.
- `jwk_fingerprints`: *string list*. A list of fingerprints (the unique identifier of the certificate) for certificate pinning and avoid man in the middle attacks. Add fingerprints in base64 format.
- `cipher_suites`: *integers list*. Override the default cipher suites. Use it if you want to enforce an even higher security standard.
- `jwk_local_ca`: *string*. Path to the certificate of the CA that verifies a secure connection when downloading the JWK. Use when not recognized by the system (e.g, self-signed certificates).

For the full list of recognized algorithms and cipher suites scroll down to the end of the document.

The following example contains every single option available:

{{< highlight JSON >}}
{
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
}
{{< /highlight >}}

## A complete running example
The [KrakenD Playground](/docs/overview/playground/) demonstrates how to protect endpoints using JWT and includes two examples ready to use:

- an integration with an external third party using a [Single Page Application from Auth0](https://auth0.com/docs/applications/spa)
- an integration with an internal identity provider service (mocked) using a symmetric key algorithm and a signer middleware.

To try it, [clone the playground](https://github.com/devopsfaith/krakend-playground) and follow the README.

## Supported hashing algorithms and cipher suites

### Hashing algorithms
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

### Cipher suites
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
