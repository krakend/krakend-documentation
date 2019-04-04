---
lastmod: 2018-11-03
date: 2018-11-03
linktitle: JWT Signing
title: JWT Signing
weight: 30
source: https://github.com/devopsfaith/krakend-jose
menu:
  documentation:
    parent: authorization
---
The JWT signing component is meant to create an **endpoint wrapper** that returns signed tokens when your application returns plain text tokens. The tokens return to the user signed with your secret key, that can be kept in the gateway or a trusted machine. With the token signing, you are in control of the private key, and you don't need to trust an external source to keep it for you.

The **JOSE component** is responsible for signing tokens.

# Requirements
You need a backend -not KrakenD- that exposes an endpoint able to **issues tokens**, and another endpoint for **refreshing tokens**.

For instance your backend could have an endpoint like `/token-issuer`, that when receives the right `POST` parameters able to identify the user returns something like this:

    curl -X POST --data '{"user":"john","pass":"doe"}' https://backend/token-issuer
    {
        "access_token": {
            "aud": "http://api.example.com",
            "iss": "https://myapp.example.com",
            "sub": "1234567890qwertyuio",
            "jti": "mnb23vcsrt756yuiomnbvcx98ertyuiop",
            "roles": ["role_a", "role_b"],
            "exp": 1735689600
        },
        "refresh_token": {
            "aud": "http://api.example.com",
            "iss": "https://myapp.example.com",
            "sub": "1234567890qwertyuio",
            "jti": "mnb23vcsrt756yuiomn12876bvcx98ertyuiop",
            "exp": 1735689600
        },
        "exp": 1735689600
    }

# Basic JWT signing
When your application knows how to issue tokens, you can sign them before passing them to the user automatically in the gateway. To achieve that, instead of publishing our internal backend that generates plain tokens under `/token-issuer`, we only expose via KrakenD a new endpoint named `/token` (choose your name). This endpoint forwards the data received in the `POST` (as chosen in the example) and returns a signed token when the backend replies.

For instance, from the plain token above we want to sign the keys `"access_token"` and `"refresh_token"` so nobody can see its contents. We need a configuration like this:

{{< highlight go "hl_lines=8-15" >}}
"endpoint": "/token",
"method": "POST",
"backend": [
{
    "host": [ "https://backend" ],
    "url_pattern": "/token-issuer"
}
],
"extra_config": {
    "github.com/devopsfaith/krakend-jose/signer": {
        "alg": "HS256",
        "kid": "sim2",
        "keys-to-sign": ["access_token", "refresh_token"],
        "jwk-url": "http://backend/jwk/symmetric.json"
    }
}
...
{{< /highlight >}}

Notice that a [JSON Web key](https://tools.ietf.org/html/rfc7517#appendix-C.1) is provided to sign the content. Generate your key and save in a secure place.

What happens here is that the user requests a `/token` to the gateway and the issuing is delegated to the backend. The response of the backend with the plain token is signed using your private JWK. And then the user receives the signed token, e.g:

    {
    "access_token": "eyJhbGciOiJIUzI1NiIsImtpZCI6InNcdTIifQ.eyJhdWQiOiJodHRwOi8vYXBpLmV4YW1wbGUuY29tIiwiZXhwIjoxNzM1Njg5NjAwLCJpf1MiOiJodHRwczovL2tyYWtlbmQuaW8iLCJqdGkiOiJtbmIyM3Zjf1J0NzU2eXVcd21uYnZjeDk4ZXJ0eXVcd3AiLCJyb2xlcyI6WyJyb2xlX2EiLCJyb2xlX2IiXSwif1ViIjoiMTIzNDU2Nzg5MHF3ZXJ0eXVcdyJ9.htgbhantGcv6zrN1i43Rl58q1sokh3lzuFgzfenI0Rk",
    "exp": 1735689600,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsImtpZCI6InNcdTIifQ.eyJhdWQiOiJodHRwOi8vYXBpLmV4YW1wbGUuY29tIiwiZXhwIjoxNzM1Njg5NjAwLCJpf1MiOiJodHRwczovL2tyYWtlbmQuaW8iLCJqdGkiOiJtbmIyM3Zjf1J0NzU2eXVcd21uMTI4NzZidmN4OThlcnR5dWlvcCIsInN1YiI6IjEyMzQ1Njc4OTBxd2VydHl1aW8ifQ.4v36tuYHe4E9gCVO-_asuXfzSzoJdoR0NJfVQdVKidw"
    }

# A complete running example
The [KrakenD Playground](/docs/overview/playground/) demonstrates how to sign tokens in the `/token` endpoint and includes an example ready to use. To try it, [clone the playground](https://github.com/devopsfaith/krakend-playground) and follow the README.


# JWT signing settings
The following settings are available to sign JWT:


- `alg`: *recognized string*. The hashing algorithm used by the issuer. Usually `RS256`.
- `jwk-url`: *string*. The URL to the JWK endpoint with the private keys used to sign the token.
- `kid`: *string*. The key ID member purpose is to match a specific key, as the jwk-url might contain several keys.
- `keys-to-sign`: *string list*. List of all the specific keys that need signing.

**Optional**:

- `full`: *boolean*. Use JSON format instead of the compact form JWT is giving.
- `disable_jwk_security`: *boolean*. When `true`, disables security of the JWK client and allows insecure connections (plain HTTP) to download the keys.
- `jwk_fingerprints`: *string list*. A list of fingerprints (the unique identifier of the certificate) for certificate pinning and avoid man in the middle attacks. Add fingerprints in base64 format.
- `cipher_suites`: *integers list*. Override the default cipher suites. Unless you have a legacy JWK, you don't need to add this value.


The following example contains every single option available:

{{< highlight go "hl_lines=6-23" >}}
"endpoints": [
    {
      "endpoint": "/token",
      "method": "POST",
      "extra_config": {
        "github.com/devopsfaith/krakend-jose/signer": {
          "alg": "HS256",
          "jwk-url": "http://backend/jwk/symmetric.json",
          "keys-to-sign": [
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
          "full": true,
          "disable_jwk_security": true
        }
      }
    }
  ]
{{< /highlight >}}