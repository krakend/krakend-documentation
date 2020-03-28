---
lastmod: 2018-11-03
date: 2018-11-03
linktitle: JWT overview
title: JSON Web Tokens overview
weight: 10
notoc: true
menu:
  documentation:
    parent: authorization
---

The **JSON Web Token** specification is an industry standard to represent claims securely between two parties. The **JWT** is an encoded JSON object that contains key-value pairs of attributes that are signed by a trusted authority.

When JWT shields a specific set of endpoints, requests to the API gateway must provide a token. Verification of the token takes place in every request, including the check of the signature and optionally the assurance that its issuer, roles, and audience are sufficient to access the endpoint. No external access is needed other than the initial load of the JWK url to validate tokens.

Only in the case that the token is valid and passes all the checks, **the user is authorized to access the endpoint** and continue with the request.

{{< note title="New to JWT?" >}}
If you are not familiar with JWT yet, read the "[Introduction to JSON Web Tokens](https://jwt.io/introduction/)"
{{< /note >}}

## KrakenD JWT implementations
KrakenD implements both [JWT signing](/docs/authorization/jwt-signing) and [JWT validation](/docs/authorization/jwt-validation) models to protect endpoints from undesired users that are not entitled to use the information, reinforcing security.

- [Sign tokens](/docs/authorization/jwt-signing) when you have no identity server yet (like a classic monolithic application with a `/login` endpoint) and let KrakenD take care of the token signing with the private key.
- [Validate tokens](/docs/authorization/jwt-validation) issued by a third party, ensuring their integrity and proper claims.


A stateless system like KrakenD **does not issue tokens**, this is the responsability of your backend or identity server.
