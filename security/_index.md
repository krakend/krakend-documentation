---
lastmod: 2024-10-09
date: 2024-10-09
linktitle:  Security Overview
title: Security Overview
description: KrakenD is built with a security-first approach in mind. Read about the security features bundled by KrakenD and the principles and design philosophy behind it
weight: -1000
notoc: false
menu:
  community_current:
    parent: "070 Security"
---
KrakenD is a software built with a **security-first philosophy**. In 2022, we became a recognized **CVE Numbering Authority (CNA)** worldwide for software distribution ([Partner page](https://www.cve.org/PartnerInformation/ListofPartners/partner/KrakenD)), and we publish CVE Records for vulnerabilities within any KrakenD software or the Lura Project (© the Linux Foundation).

{{< button-group >}}
{{< button url="/categories/security/" text="Security Advisories" >}}<svg viewBox="0 0 22 22" xmlns="http://www.w3.org/2000/svg">
  <g stroke="currentColor" fill="none" fill-rule="evenodd">
    <path d="M15.906 10.358h-10.5c-1.087 0-1.968.862-1.968 1.925V18.7c0 1.063.88 1.925 1.968 1.925h10.5c1.088 0 1.969-.862 1.969-1.925v-6.417c0-1.063-.881-1.925-1.969-1.925z" stroke-width="1.375" stroke-linecap="round" stroke-linejoin="round"></path>
    <path d="M6.063 10.358V5.867c0-2.481 2.056-4.492 4.593-4.492s4.594 2.011 4.594 4.492v4.491M3.438 15.492h14.438" stroke-width="1.375" stroke-linecap="round" stroke-linejoin="round"></path>
    <path d="M6.719 13.246a.325.325 0 0 1-.328-.321c0-.177.147-.32.328-.32M6.719 13.246a.325.325 0 0 0 .328-.321.325.325 0 0 0-.328-.32" stroke-width="1.375"></path>
    <path d="M6.719 18.38a.325.325 0 0 1-.328-.322c0-.177.147-.32.328-.32M6.719 18.38a.325.325 0 0 0 .328-.322.325.325 0 0 0-.328-.32" stroke-width="1.369"></path>
  </g>
</svg>
{{< /button >}}
{{< button url="/security-policy/" type="inversed" >}}Security Policy{{< /button >}}
{{< /button-group >}}

## Secure by design
At KrakenD, security is not just an add-on; it's a **design principle baked into every component**. The [Zero-trust design](/docs/design/zero-trust/) is the foundational philosophy. From blocking unauthorized access to rejecting untrusted traffic by default or even not logging sensitive data, KrakenD ensures a minimal attack surface by enforcing strict controls over headers, parameters, and tokens.

Our *Security Program Policy and Incident Response Plan* have the following principles:

- **Secure Development and Proactive Threat Detection**: To ensure that KrakenD is secure, we conduct continuous automated code analysis, vulnerability assessments, and other security measures integrated into the CI/CD pipeline.
- **Software Integrity**: To protect the codebase's and software's integrity by enforcing security measures that prevent unauthorized changes, reduce human error, and mitigate potential security vulnerabilities in real time.
- **Rapid Incident Response**: To ensure a quick and effective response to security incidents and minimize their impact through defined protocols for containment, eradication, recovery, and post-incident analysis.
- **Compliance with Industry Standards**: To ensure KrakenD's software adheres to industry standards and security frameworks, such as OWASP best practices, and complies with regulatory requirements for enterprises.
- **Enterprise-Ready Security**: To provide a robust security framework suitable for large-scale enterprise deployment, ensuring that all software produced by KrakenD is safe, scalable, and reliable for its enterprise customers.

Below are the categories in which security is more obvious. Although this is not a complete list, it provides you with a place to start exploring our documentation.

## Authentication and Authorization
API authentication and authorization are key to any secured API. KrakenD has mechanisms such as [JWT validation](/docs/authorization/jwt-validation/), [JWT signing](/docs/authorization/jwt-signing/), [OAuth2 Client Credentials](/docs/authorization/client-credentials/) or [API keys](/docs/enterprise/authentication/api-keys/) {{< badge color="denim" >}}Enterprise{{< /badge >}} to name a few examples.

Authorization allows you to implement Role-based (RBAC) and attribute-based access control (ABAC) policies.

In addition, if you need to invalidate legitimate tokens that are still within a valid TTL, KrakenD supports [JWT token revocation using bloom filters](/docs/authorization/revoking-tokens/) and [centralized token revocation servers](/docs/enterprise/authentication/revoke-server/), ensuring revoked tokens are immediately invalidated across all KrakenD nodes.

## Encryption and Secure Communication
The gateway supports [TLS](/docs/service-settings/tls/) for traffic comming from consumers (server) and also between KrakenD and your services (client). It defaults to TLS 1.3 unless downgraded by config.

For business-to-business authentication, [Mutual TLS (mTLS)](/docs/authorization/mutual-authentication/) creates a secure and exclusive channel based on trusted certificates.

Governments can also get a Docker container with [FIPS 140-2 validated cryptography](/docs/enterprise/security/fips-140/) {{< badge color="denim" >}}Enterprise{{< /badge >}} for compliance with their regulations.

## Data protection
Showing the right data or allowing limited access is key on any API. In addition to blocking users without enough privileges to consume data, you can apply [data filtering and manipulation](/docs/enterprise/backends/data-manipulation/) or even [masking of data](/docs/enterprise/endpoints/content-replacer/) {{< badge color="denim" >}}Enterprise{{< /badge >}}

In addition, to prevent malicious or malformed requests, KrakenD allows you to [validate the payload](/docs/endpoints/JSON-schema/) of requests against a JSON schema before it reaches your service. But it also works the other way around: you can also [validate responses](/docs/enterprise/endpoints/response-schema-validator/) {{< badge color="denim" >}}Enterprise{{< /badge >}} of your services against a schema and decide whether is worth or not returning it to the end user.

Finally, the [Security policy engine](/docs/enterprise/security-policies/) is designed to enforce complex business logic based on real-time evaluation of requests, responses, and tokens.

## Traffic Control
**[API Throttling](/docs/enterprise/throttling/)** is a dragon of many heads. You might want to limit the throughput your users do against your API with one of the many rate-limiting strategies: [per-service](/docs/enterprise/service-settings/service-rate-limit/), [per-tier](/docs/enterprise/service-settings/tiered-rate-limit/) (both {{< badge color="denim" >}}Enterprise{{< /badge >}})
, [per-endpoint](/docs/endpoints/rate-limit/), [per-user](/docs/endpoints/rate-limit/#client-rate-limiting-client_max_rate), or [per-proxy](/docs/backends/rate-limit/).

Another key security component is the [Circuit Breaker](/docs/backends/circuit-breaker/), which automatically blocks calls to failing backends, **preventing cascading failures** and reducing the load on a suffering system.

Then, depending on your environment you might want to enable [IP Filtering](/docs/enterprise/throttling/ipfilter/) or [GeoIP filtering](/docs/enterprise/endpoints/geoip/) to restrict API traffic based on IP addresses, CIDR ranges, or geography (both are {{< badge color="denim" >}}Enterprise{{< /badge >}}
), [Bot detection](/docs/throttling/botdetector/),or enable conditional requests with [Conditional Expression Language](/docs/endpoints/common-expression-language-cel/) (CEL) or Security Policies (also {{< badge color="denim" >}}Enterprise{{< /badge >}}
).

## HTTPS Security and OWASP Recommendations
KrakenD follows **OWASP best practices** and security recommendations, incorporating several protections by just declaring the security component:
- [Host Restriction](/docs/service-settings/security/#restrict-connections-by-host): Restrict connections by host, defining a list of backends that the API gateway can communicate with.
- [Cross-Origin Resource Sharing (CORS)](/docs/service-settings/cors/) lets you control and limit which domains can access APIs, protecting against cross-origin attacks.
- [HTTP Strict Transport Security (HSTS)](/docs/service-settings/security/#http-strict-transport-security-hsts) makes sure that all interactions with the gateway use HTTPS, mitigating protocol downgrade attacks.
- **Public Key Pinning**: To [prevent certificate forgery](/docs/service-settings/security/#http-public-key-pinning-hpkp), HPKP allows you to "pin" a public key, ensuring clients connect to the intended service.
- **Clickjacking Protection**: To activate frame-busting mechanisms by [configuring X-Frame-Options headers](/docs/service-settings/security/#clickjacking-protection).
- **Cross-Site Scripting (XSS) Protection**: Mitigate XSS attacks by adding relevant security headers like [X-XSS-Protection](/docs/service-settings/security/#cross-site-scripting-xss-protection), protecting clients from malicious script injections.
- **MIME Sniffing Prevention** prevents browsers from [MIME sniffing](/docs/service-settings/security/#mime-sniffing-prevention) and interpreting files as a different content type than declared by using the X-Content-Type-Options header.

## Monitoring, Auditing, and Logging
Logging and Monitoring, like OpenTelemetry, Prometheus, New Relic, Datadog, and other integrations, ensure that audit trails are available for all requests and responses, which is crucial for forensics and compliance.

Another part directly related to security is the [automatic audit of configuration](/docs/configuration/audit/), a step in your build process that checks whether your configuration has security problems or it can be improved before going live.

## Tested by many
KrakenD's security is strengthened by the fact that **it is tested by thousands of servers every day** across diverse environments, geographies, and use cases. This extensive usage (approx. 2 million servers/month) means that potential vulnerabilities are identified and addressed quickly, as real-world scenarios expose the system to a wide range of security challenges. Continuous feedback from a large community of developers ensures that KrakenD remains resilient to new threats, benefits from community-driven improvements, and maintains robust security practices. This collective testing approach makes KrakenD more secure and reliable over time.

## No data storage
As KrakenD operates as a stateless gateway, only processes data in transit and **does not store any information**. Since KrakenD does not retain user data, logs, or any sensitive information, it reduces the risk of data breaches or unauthorized access. This design ensures that all data flows securely through the system without lingering in any storage, making KrakenD inherently more secure and compliant with heavy data privacy regulations (banking, health, insurance, etc), as it minimizes the exposure of sensitive information in an eventual breach.
