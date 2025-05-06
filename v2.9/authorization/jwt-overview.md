---
lastmod: 2018-11-03
old_version: true
date: 2018-11-03
linktitle: JWT overview
title: JWT Overview - Authorization
description: Gain an overview of JSON Web Tokens (JWT) and learn how to implement JWT-based authorization with KrakenD API Gateway for secure API access
weight: 10
notoc: true
menu:
  community_v2.9:
    parent: "080 Authentication & Authorization"
---

The **JSON Web Token** specification is an industry standard to represent claims securely between two parties. The **JWT** is a `base64` encoded JSON object that contains key-value pairs of attributes that are signed by a trusted authority.

When JWT shields a specific set of endpoints, requests to the API gateway must provide a token. Verification of the token takes place in every request, including the check of the signature and optionally the assurance that its issuer, roles, and audience are sufficient to access the endpoint. No external access is needed other than the initial load of the JWK URL to validate tokens.

Only in the case that the token is valid and passes all the checks, **the user is authorized to access the endpoint** and continue with the request.

## KrakenD JWT implementations
KrakenD implements both [JWT signing](/docs/v2.9/authorization/jwt-signing/) and [JWT validation](/docs/v2.9/authorization/jwt-validation/) models to protect endpoints from undesired users that are not entitled to use the information, reinforcing security.

- [Sign tokens](/docs/v2.9/authorization/jwt-signing/) when you have no identity server yet (like a classic monolithic application with a `/login` endpoint) and let KrakenD take care of the token signing with the private key.
- [Validate tokens](/docs/v2.9/authorization/jwt-validation/) issued by a third party or the [JWT signing middleware](/docs/v2.9/authorization/jwt-signing/), ensuring their integrity and proper claims.


A stateless system like KrakenD **does not issue tokens**, this is the responsibility of your backend or identity server.

## Key concepts
The **JSON Web Token** carries the information your end-users pass to the system to be recognized as legitimate users with other metadata.

KrakenD uses **standard JWT tokens** to protect endpoints, using JSON Web Signature (**JWS**), to check the tokens' digital signature integrity of the contained claims and defending against attacks using tampered tokens.

A JWT token is a `base64` encoded string with the structure `header.payload.signature`.

A typical request to an endpoint requiring JWT validation includes a `Bearer` in the `Authorization` header:

{{< terminal >}}
GET /resource HTTP/1.1
Host: krakend.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIXVCJ9.(truncated).ktIOfzak2ekD7IrCa9-UiO4QA
{{< /terminal >}}

Or instead, you can send the token **inside a cookie** (see [`cookie_key`](/docs/v2.9/authorization/jwt-validation/#jwt-validation-settings)).


All tokens transmitted between users and KrakenD have to be signed using **JWS** to ensure they are legitimate and not forged by an attacker. JWS represents digitally signed content using JSON data structures that are base64url encoded using the format `header.payload.signature`.

Finally, KrakenD needs to retrieve the keys from the trusted authority (your Identity Provider) that let the system validate the signature. These keys transmit between KrakenD and the IdP using the **JWK** format, a JSON object representing a set of cryptographic keys. Objects will use one or another algorithm depending on the system and implementation in your IdP. **JWA** represents the set of algorithms you can use to sign your tokens.

The introduction above is very superficial; the recommended read is the RFC:

- **JWT**: [Definition of tokens: structure and composition of header and payload](https://tools.ietf.org/html/rfc7519)
- **JWS** [Signature](https://tools.ietf.org/html/rfc7515)
- **JWK** [Key transmission](https://tools.ietf.org/html/rfc7517)
- **JWA** [Definition of cyphering and signing algorithms](https://tools.ietf.org/html/rfc7518)
- **JWE** is not supported by KrakenD (Premise: Sensitive data should not be transmitted using tokens).

{{< note title="New to JWT?" >}}
If you are not familiar with JWT yet, read the "[Introduction to JSON Web Tokens](https://jwt.io/introduction/)"
{{< /note >}}
