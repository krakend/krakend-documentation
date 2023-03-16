---
lastmod: 2023-03-13
old_version: true
date: 2023-03-13
description: The zero-trust policy is a deny by default software design choice, where the exposed surface is limited to what it's explictly declared.
linktitle: Zero-trust policy
title: Zero-trust policy
weight: 10
menu:
  community_v2.2:
    parent: "170 Design principles"
---

Real-world API deployments suffer attacks every day, even if you don't notice it. Where there is an accessible server, there is malicious activity.

The Zero Trust security is a software architecture design choice to **deny by default any activity unless specifically allowed**. This type of policy is very secure, but usually adds a lot of burden on infrastructure administrators. KrakenD offers a balance of tools and default secure choices to ease the administration while keeping the software secure.

## Zero Trust pillars

1. Explicit declaration
2. Least-privilege access
3. Assume breach



Nothing behind the corporate firewall is safe. Nothing behind the gateway is safe. The Zero Trust model assumes breach and verifies each request as it originates from an open network. Regardless of where the request originates or what resource it accesses, Zero Trust teaches us to "never trust, always verify."

Every access request with tokens is fully authorized, and decrypted before granting access. Segmentation by multiple criteria and least-privilege access principles are applied to minimize lateral movement.


KrakenD assumes the following behaviors when serving an API:

- No endpoints exposed unless explicitly declared:
  - No possible scan of the upstream services
  - No Zombie APIs as all routes are typed. No possibility to leave unnoticed endpoints behind
- No header, query string, or cookie forwarding unless explictly declared which ones:
  - Injections limited because only what is explicitly declared is the atack surface
- TLS defaults to TLS v1.3, the most secure, and rejects older versions
- OWASP recommendations applied risks mitigated, including but not limited to the Top 10:
    - [A01:2021-Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
    - [A02:2021-Cryptographic Failures](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)
    - [A03:2021-Injection](https://owasp.org/Top10/A03_2021-Injection/)
    - [A04:2021-Insecure Design](https://owasp.org/Top10/A04_2021-Insecure_Design/)
    - [A05:2021-Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
    - [A06:2021-Vulnerable and Outdated Components](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/)
    - [A07:2021-Identification and Authentication Failures](https://owasp.org/Top10/A07_2021-Identification_and_Authentication_Failures/)
    - [A08:2021-Software and Data Integrity Failures](https://owasp.org/Top10/A08_2021-Software_and_Data_Integrity_Failures/)
    - [A09:2021-Security Logging and Monitoring Failures](https://owasp.org/Top10/A09_2021-Security_Logging_and_Monitoring_Failures/)
    - [A10:2021-Server-Side Request Forgery](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/)
