---
aliases:
- /features/security/
lastmod: 2016-09-30
date: 2016-09-30
toc: true
linktitle: Security
title: Security
weight: 40
menu:
  main:
    parent: features
---

KrakenD has implemented some security strategies

# General security

## Restrict connections by host

Define a whitelist of hosts the KrakenD should accept requests to.

When a request hits KrakenD, it will confirm if the value of the `Host` HTTP header is in the whitelist. If so, it will further process the request. If the host is not in the whitelist, KrakenD will simply reject the request.

## Clickjacking protection

KrakenD follow the OWASP's recommendations by adding a frame-breaking strategy.

You can add a `X-Frame-Options` header with the value of `DENY` (default behaviour) or even set your custom value.

Check the [OWASP Clickjaking cheat sheet](https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet#X-Frame-Options_Header_Types) for more details about the header and its recommended values.

## MIME-Sniffing prevention

Enabling this feature will prevent the user's browser from interpreting files as something else than declared by the content type in the HTTP headers.

## Cross-site scripting (XSS) protection

This feature enables the [Cross-site scripting (XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)) filter in the user's browser.

# HTTPS

## HTTP Strict Transport Security (HSTS)

OWASP defines the HSTS as

> HTTP Strict Transport Security (HSTS) is a web security policy mechanism which helps to protect websites against protocol downgrade attacks and cookie hijacking. It allows web servers to declare that web browsers (or other complying user agents) should only interact with it using secure HTTPS connections, and never via the insecure HTTP protocol. HSTS is an IETF standards track protocol and is specified in RFC 6797. A server implements an HSTS policy by supplying a header (Strict-Transport-Security) over an HTTPS connection (HSTS headers over HTTP are ignored).

## HTTP Public Key Pinning (HPKP)

OWASP defines the HPKP as

> HTTP Public Key Pinning (HPKP) is a security mechanism which allows HTTPS websites to resist impersonation by attackers using mis-issued or otherwise fraudulent certificates. (For example, sometimes attackers can compromise certificate authorities, and then can mis-issue certificates for a web origin.).

**This feature must be used with caution, because there is a risk that hosts may make themselves unavailable by pinning to a set of public key hashes that becomes invalid.**

# OAuth2

KrakenD supports the client credentials grant.

Use this feature if you need to authorize the KrakenD to access your backend services.