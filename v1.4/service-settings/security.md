---
lastmod: 2021-05-02
old_version: true
date: 2016-09-30
linktitle: Security
title: Security
weight: 40
menu:
  community_v1.4:
    parent: "030 Service Settings"
meta:
  since: 0.4
  source: https://github.com/devopsfaith/krakend-httpsecure
  namespace:
  - github_com/devopsfaith/krakend-httpsecure
  scope:
  - service
---

KrakenD has implemented several security strategies, controlled via [krakend-httpsecure](https://github.com/devopsfaith/krakend-httpsecure). To enable them you only need to add the `extra_config` at service (root) level.

The following example describes the options explained later in this article:

    "extra_config": {
      "github_com/devopsfaith/krakend-httpsecure": {
        "allowed_hosts": [
          "host.known.com:443"
        ],
        "ssl_proxy_headers": {
          "X-Forwarded-Proto": "https"
        },
        "ssl_redirect": true,
        "ssl_host": "ssl.host.domain",
        "ssl_port": "443",
        "ssl_certificate": "/path/to/cert",
        "ssl_private_key": "/path/to/key",
        "sts_seconds": 300,
        "sts_include_subdomains": true,
        "frame_deny": true,
        "custom_frame_options_value": "ALLOW-FROM https://example.com",
        "hpkp_public_key": "pin-sha256=\"base64==\"; max-age=expireTime [; includeSubDomains][; report-uri=\"reportURI\"]",
        "content_type_nosniff": true,
        "browser_xss_filter": true,
        "content_security_policy": "default-src 'self';"
      }

See below the different options described in this configuration file.

## General security

### Restrict connections by host
Use `allowed_hosts`

Define a list of hosts that KrakenD should accept requests to.

When a request hits KrakenD, it will confirm if the value of the `Host` HTTP header is in the list. If so, it will further process the request. If the host is not in the allowed hosts list, KrakenD will simply reject the request.

The list must contain the fully qualified domain names that are allowed, along with the origin port. When the list is empty accepts any host.

### Clickjacking protection
Use `frame_deny`

KrakenD follow the OWASP's recommendations by adding a frame-breaking strategy.

You can add an `X-Frame-Options` header using `custom_frame_options_value` with the value of `DENY` (default behavior) or even set your custom value.

Check the [OWASP Clickjacking cheat sheet](https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet#X-Frame-Options_Header_Types) for more details about the header and its recommended values.

### MIME-Sniffing prevention
Use `content_type_nosniff`

Enabling this feature will prevent the user's browser from interpreting files as something else than declared by the content type in the HTTP headers.

### Cross-site scripting (XSS) protection
Use `browser_xss_filter`

This feature enables the [Cross-site scripting (XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)) filter in the user's browser.

## HTTPS

### HTTP Strict Transport Security (HSTS)
Use `sts_seconds`

OWASP defines the HSTS as

> HTTP Strict Transport Security (HSTS) is a web security policy mechanism which helps to protect websites against protocol downgrade attacks and cookie hijacking. It allows web servers to declare that web browsers (or other complying user agents) should only interact with it using secure HTTPS connections, and never via the insecure HTTP protocol. HSTS is an IETF standards track protocol and is specified in RFC 6797. A server implements an HSTS policy by supplying a header (Strict-Transport-Security) over an HTTPS connection (HSTS headers over HTTP are ignored).

Enable this policy by setting the max-age of the Strict-Transport-Security header. Setting to `0` disables HSTS. Use the `sts_seconds` setting.

### HTTP Public Key Pinning (HPKP)
Use `hpkp_public_key`

OWASP defines the HPKP as

> HTTP Public Key Pinning (HPKP) is a security mechanism which allows HTTPS websites to resist impersonation by attackers using mis-issued or otherwise fraudulent certificates. (For example, sometimes attackers can compromise certificate authorities, and then can mis-issue certificates for a web origin.).

**This feature must be used with caution because there is a risk that hosts may make themselves unavailable by pinning to a set of public key hashes that becomes invalid.**

## OAuth2

KrakenD supports the client credentials grant.

Use this feature if you need to authorize the KrakenD to access your backend services.

See the specific docs for [OAuth2 Client Credentials](/docs/v1.4/authorization/client-credentials/)
