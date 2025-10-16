---
lastmod: 2022-01-31
old_version: true
old_version: true
date: 2016-09-30
linktitle: HTTP Security
title: HTTP Security Considerations
description: KrakenD's HTTP Security component enhances API protection by enabling features like Clickjacking protection, MIME-Sniffing prevention, and XSS protection, and more
weight: 40
menu:
  community_v2.10:
    parent: "070 Security"
meta:
  since: v0.4
  source: https://github.com/krakend/krakend-httpsecure
  namespace:
  - security/http
  scope:
  - service
---

KrakenD has implemented several security strategies, controlled via the `security/http` component. To enable them you only need to add its namespace `security/http` at the `extra_config` in the root level of the configuration.

The following configuration describes all possible options:

```json
{
    "version": 3,
    "extra_config": {
      "security/http": {
        "allowed_hosts": [
          "host.known.com:443"
        ],
        "ssl_proxy_headers": {
          "X-Forwarded-Proto": "https"
        },
        "host_proxy_headers":[
          "X-Forwarded-Hosts"
        ],
        "ssl_redirect": true,
        "ssl_host": "ssl.host.domain",
        "sts_seconds": 300,
        "sts_include_subdomains": true,
        "frame_deny": true,
        "referrer_policy": "same-origin",
        "custom_frame_options_value": "ALLOW-FROM https://example.com",
        "hpkp_public_key": "pin-sha256=\"base64==\"; max-age=expireTime [; includeSubDomains][; report-uri=\"reportURI\"]",
        "content_type_nosniff": true,
        "browser_xss_filter": true,
        "content_security_policy": "default-src 'self';",
        "is_development": false
      }
}
```

See below the different options described in this configuration file.

{{< schema version="v2.10" data="security/http.json" >}}


### Restrict connections by host
Use `allowed_hosts`

Define a list of hosts that KrakenD should accept requests to.

When a request hits KrakenD, it will confirm if the value of the `Host` HTTP header is in the list. If so, it will further process the request. If the host is not in the allowed hosts list, KrakenD will simply reject the request.

The list must contain the fully qualified domain names that are allowed, along with the origin port. When the list is empty accepts any host.

### Clickjacking protection
KrakenD follow the OWASP's recommendations by adding a frame-breaking strategy.

Use `frame_deny` together with `custom_frame_options_value`

You can add an `X-Frame-Options` header using `custom_frame_options_value` with the value of `DENY` (default behavior) or even set your custom value.

Check the [OWASP Clickjacking cheat sheet](https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet#X-Frame-Options_Header_Types) for more details about the header and its recommended values.

### MIME-Sniffing prevention
Use `content_type_nosniff`

Enabling this feature will prevent the user's browser from interpreting files as something else than declared by the content type in the HTTP headers.

### Cross-site scripting (XSS) protection
Use `browser_xss_filter`

This feature enables the [Cross-site scripting (XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)) filter in the user's browser.

### Content-Security-Policy
Related to XSS protection there is the HTTP Content-Security-Policy response header, which allows you to control resources the user agent is allowed to load for a given page.

Use `content_security_policy` (*string*) to set your policy. E.g.: `default-src 'self';`

## HTTPS

### HTTP Strict Transport Security (HSTS)
OWASP defines the HSTS as

> HTTP Strict Transport Security (HSTS) is a web security policy mechanism which helps to protect websites against protocol downgrade attacks and cookie hijacking. It allows web servers to declare that web browsers (or other complying user agents) should only interact with it using secure HTTPS connections, and never via the insecure HTTP protocol. HSTS is an IETF standards track protocol and is specified in RFC 6797. A server implements an HSTS policy by supplying a header (Strict-Transport-Security) over an HTTPS connection (HSTS headers over HTTP are ignored).

- Use `sts_seconds` (*integer*): Enable this policy by setting the max-age of the Strict-Transport-Security header. Setting to `0` disables HSTS. Use the `sts_seconds` setting.
- Use `sts_include_subdomains` (*bool*): Set to true when you want the `includeSubdomains` be appended to the Strict-Transport-Security header.

### HTTP Public Key Pinning (HPKP)
Use `hpkp_public_key`

OWASP defines the HPKP as

> HTTP Public Key Pinning (HPKP) is a security mechanism which allows HTTPS websites to resist impersonation by attackers using mis-issued or otherwise fraudulent certificates. (For example, sometimes attackers can compromise certificate authorities, and then can mis-issue certificates for a web origin.).

**This feature must be used with caution because there is a risk that hosts may make themselves unavailable by pinning to a set of public key hashes that becomes invalid.**

## OAuth2

KrakenD supports the client credentials grant.

Use this feature if you need to authorize the KrakenD to access your backend services.

See the specific docs for [OAuth2 Client Credentials](/docs/v2.11/v2.10/authorization/client-credentials/)
