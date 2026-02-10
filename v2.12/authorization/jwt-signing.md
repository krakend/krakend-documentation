---
lastmod: 2022-10-24
old_version: true
date: 2018-11-03
linktitle: JWT Signing
title: JWT Signing
description: KrakenD's JWT signing wraps your login endpoint, signing payloads with your secret key. Perfect for monolith migrations or setups without an OAuth server.
weight: 30
images: ["/images/krakend-signer-flow.png"]
dark_header_image: true
menu:
  community_v2.12:
    parent: "080 Authentication & Authorization"
meta:
  source: https://github.com/krakend/krakend-jose
  namespace:
  - auth/signer
  scope:
  - endpoint
  log_prefix:
  - "[ENDPOINT: /foo][JWTSigner]"
---

The JWT signing component creates a **wrapper for your existing login endpoint** that signs with your secret key the selected fields of the backend payload right before returning the content to the end-user.

The primary usage for this component is in **migrations from monolith to microservices**, or in ecosystems where there is no Identity/OAuth server yet, as it allows the immediate adoption of signed JSON Web Tokens without the need to implement a new service.

## How does it work
KrakenD relies in your existing login functionality and does all the heavy-lifting of the cryptography so you can focus on validating the user and password.

Your backend needs to implement a **login endpoint** that after validating the username and password **it returns a JSON reponse**, and optionally another endpoint for refreshing tokens. For example:

```json
{
    "access_token": {
        "aud": "http://api.example.com",
        "iss": "https://krakend.io",
        "sub": "1234567890qwertyuio",
        "jti": "mnb23vcsrt756yuiomnbvcx98ertyuiop",
        "roles": ["role_a", "role_b"],
        "exp": 1735689600
    },
    "refresh_token": {
        "aud": "http://api.example.com",
        "iss": "https://krakend.io",
        "sub": "1234567890qwertyuio",
        "jti": "mnb23vcsrt756yuiomn12876bvcx98ertyuiop",
        "exp": 1735689600
    },
    "exp": 1735689600
}
```

The response payload has the [structure of a JWT token](https://www.rfc-editor.org/rfc/rfc7519#section-4.1) which contains fields like the `sub`ject (a.k.a id_user), the `jti` (a *uniqid* for instance), or the `exp`iration of the token to name a few.

When KrakenD receives this JSON payload, it signs the selected group of claims with your secret key. The secret key can be kept in the gateway or URL-downloaded from a trusted machine that you own. With the token signing, you are in control of the private key, and you don't need to trust an external service to keep it for you.

### Example
For instance, your backend could have an endpoint like `/token-issuer` that when receives the right combination of username and password via `POST` can identify the user and, instead of setting the session, returns an output like this:

{{< terminal title="Example">}}
curl -X POST --data '{"user":"john","pass":"doe"}' https://your-backend/token-issuer
{
    "access_token": {
        "aud": "https://your.krakend.io",
        "iss": "https://your-backend",
        "sub": "1234567890qwertyuio",
        "jti": "mnb23vcsrt756yuiomnbvcx98ertyuiop",
        "roles": ["role_a", "role_b"],
        "exp": 1735689600
    },
    "refresh_token": {
        "aud": "https://your.krakend.io",
        "iss": "https://your-backend",
        "sub": "1234567890qwertyuio",
        "jti": "mnb23vcsrt756yuiomn12876bvcx98ertyuiop",
        "exp": 1735689600
    },
    "exp": 1735689600
}
{{< /terminal >}}


Besides these example keys, the payload can contain any other elements you might need.

If you come from a classic login system, based on cookie sessions, you'll realize that adapting your `/login` to this output is straightforward. See [how to generate a token](#how-to-generate-a-jwt-token) at the end of the document for more details.


## JWT signing settings
The following settings are available to sign JWT:

{{< highlight json "hl_lines=7-24" >}}
{
  "endpoints": [
    {
      "endpoint": "/token",
      "method": "POST",
      "extra_config": {
        "auth/signer": {
          "alg": "HS256",
          "jwk_url": "http://your-backend/jwk/symmetric.json",
          "keys_to_sign": [
            "access_token",
            "refresh_token"
          ],
          "kid": "sim2",
          "cipher_suites": [
            5,
            10
          ],
          "jwk_fingerprints": [
            "S3Jha2VuRCBpcyB0aGUgYmVzdCBnYXRld2F5LCBhbmQgeW91IGtub3cgaXQ=="
          ],
          "full": false,
          "disable_jwk_security": false
        }
      }
    }
  ]
}
{{< /highlight >}}

The example above contains every single option available, but you don't need them all. See them explained below:

{{< schema version="v2.12" data="auth/signer.json" >}}

## Basic JWT signing
Your backend application knows how to issue tokens now, so the gateway can sign them before passing to the user. To achieve that, instead of publishing our internal backend that generates plain tokens under `/token-issuer`, we only expose via KrakenD a new endpoint named `/token` (choose your name). This endpoint forwards the data received in the `POST` (as selected in the example) and returns a signed token when the backend replies.

For instance, from the plain token above we want to sign the keys `"access_token"` and `"refresh_token"` so nobody can modify its contents. We need a configuration like this:

```json
{
  "endpoint": "/login",
  "method": "GET",
  "backend": [
    {
      "url_pattern": "/login",
      "host": ["http://backend-url"]
    }
  ],
  "extra_config": {
    "auth/signer": {
      "alg": "HS256",
      "kid": "sim2",
      "keys_to_sign": ["access_token", "refresh_token"],
      "jwk_local_path": "jwk_private_key.json",
      "disable_jwk_security": true
    }
  }
}
```

The content of `jwk_private_key.json` used in this example [is here](https://github.com/krakend/playground-community/blob/master/data/jwk/symmetric.json).


Notice that we have added a file under `jwk_local_path` which is a [JSON Web key](https://tools.ietf.org/html/rfc7517#appendix-C.1) (could also be hosted via `jwk_url`).

The example adds a `disable_jwk_security` flag because downloading the file from `jwk_local_path` does not use the HTTP protocol.


What happens here is that the user requests a `/token` to the gateway and the issuing is delegated to the backend. The response of the backend with the plain token is signed using your private JWK. And then the user receives the signed token, e.g:

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsImtpZCI6InNcdTIifQ.eyJhdWQiOiJodHRwOi8vYXBpLmV4YW1wbGUuY29tIiwiZXhwIjoxNzM1Njg5NjAwLCJpf1MiOiJodHRwczovL2tyYWtlbmQuaW8iLCJqdGkiOiJtbmIyM3Zjf1J0NzU2eXVcd21uYnZjeDk4ZXJ0eXVcd3AiLCJyb2xlcyI6WyJyb2xlX2EiLCJyb2xlX2IiXSwif1ViIjoiMTIzNDU2Nzg5MHF3ZXJ0eXVcdyJ9.htgbhantGcv6zrN1i43Rl58q1sokh3lzuFgzfenI0Rk",
    "exp": 1735689600,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsImtpZCI6InNcdTIifQ.eyJhdWQiOiJodHRwOi8vYXBpLmV4YW1wbGUuY29tIiwiZXhwIjoxNzM1Njg5NjAwLCJpf1MiOiJodHRwczovL2tyYWtlbmQuaW8iLCJqdGkiOiJtbmIyM3Zjf1J0NzU2eXVcd21uMTI4NzZidmN4OThlcnR5dWlvcCIsInN1YiI6IjEyMzQ1Njc4OTBxd2VydHl1aW8ifQ.4v36tuYHe4E9gCVO-_asuXfzSzoJdoR0NJfVQdVKidw"
}
```

## How to convert PEM to JWKS ?
Krakend uses `jose` library, so if you use your own PEM keys for signing you need to use the following steps to [convert your PEM file to JWKS](https://web3auth.io/docs/auth-provider-setup/byo-jwt-providers#how-to-convert-pem-to-jwks)

Notice: if you are using [asymmetric algorithms](https://auth0.com/blog/navigating-rs256-and-jwks) and want to use gateway singing and verification simultaneously, you need to use the following keys with the same `kid`:
-  **private key jwks for signing**
-  **public key jwks for verification**


## How to generate a JWT token
Essentially, what you need to adopt JWT in your backend is to adapt your existing `/login` function (maybe passing an additional `?token=true` flag), so when a user logs in, instead of setting the session in a cookie, you return the JSON Web Token for KrakenD to sign.

The token is no more than a JSON output adhering to the [JWT standard](https://tools.ietf.org/html/rfc7519).

There are a lot of **open source libraries to generate JWT tokens** in all major languages. Use them or write the JSON output directly with a simple template.

Here is a [dummy token](https://github.com/krakend/playground-community/blob/master/data/token.json) for you to check how it looks like.

## Live running example
The [KrakenD Playground](/docs/v2.12/overview/playground/) demonstrates how to sign tokens in the `/token` endpoint and includes an example ready to use. To try it, [clone the playground](https://github.com/krakend/playground-community) and follow the README.
