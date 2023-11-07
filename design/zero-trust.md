---
lastmod: 2023-04-13
date: 2023-04-13
description: The zero-trust security is a deny by default software design choice, where the exposed surface is limited to what it's explictly declared.
linktitle: Zero-trust security
title: "Zero Trust Design: Secure API Architecture"
weight: 10
menu:
  community_current:
    parent: "170 Design principles"
---
Real-world API deployments suffer daily attacks, even if you don't notice them. It doesn't matter if you have your infrastructure behind a delimited perimeter. Where there is a point of access, there is malicious activity.

The Zero Trust security is a software architecture design choice to **deny by default unless specifically allowed**. In zero trust, you verify all requests, regardless of origin, and prove you can trust them. KrakenD both **evaluates** and **enforces** rules.

## The pillars of zero-trust

1. **Explicit declaration**: Do not let pass parameters, headers, query strings, etc., unless you have explicitly declared it. Don't assume any open defaults; write any behavior in the configuration.
2. **Principle of least authority (PoLA)**: Assign to your users only those privileges which are essential to perform its intended function. For example, a user account for creating deals in a sales department does not need to create invoices: hence, it has rights only to create deals, and any other privileges are blocked.
3. **Assume breach**: Users' credentials might be stolen and servers compromised. Minimize the attacking surface by segmenting access, work only with end-to-end encryption, and visualize activity in the dashboards.

## Zero-trust assumptions
KrakenD assumes the following behaviors when serving an API:

- All endpoints protected by tokens are strictly checked
  - No negotiation of algorithms possible
  - Do not trust keys from insecure protocols (HTTP)
  - No possibility of using expired tokens or incorrectly signed
  - Rules, audience, issuers, claims and business logic can be enforced
- No endpoints exposed unless explicitly declared. It provides two immediate benefits:
  - **No possible scan** of the upstream services
  - **No zombie APIs** as all routes are typed. No possibility of leaving unnoticed endpoints behind
- No header, query string, or cookie forwarding unless explicitly declared which ones:
  - Injections are limited because only what is explicitly declared is the attack surface
- TLS defaults to TLS v1.3, the most secure, and rejects older versions
- Multilayered rate limit (by service, by the user, by upstream)
- CORS configuration denies API access from untrusted origins even when credentials are OK.
- All software is designed with OWASP recommendations in mind to help mitigate risks. Including but not limited to the **Top 10**:
    - Broken Access Control
    - Cryptographic Failures
    - Injection
    - Insecure Design
    - Security Misconfiguration
    - Vulnerable and Outdated Components
    - Identification and Authentication Failures
    - Software and Data Integrity Failures
    - Security Logging and Monitoring Failures
    - Server-Side Request Forgery
