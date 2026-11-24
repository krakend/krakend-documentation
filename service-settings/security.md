---
lastmod: 2026-11-24
date: 2016-09-30
linktitle: HTTP Security
title: HTTP Security Considerations
description: KrakenD's HTTP Security component enhances API protection by enabling features like Clickjacking protection, MIME-Sniffing prevention, and XSS protection, and more
weight: 40
menu:
  community_current:
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
    "version": 4,
    "extra_config": {
      "security/http": {
        "allowed_hosts": [
          "host.known.com"
        ],
        "ssl_proxy_headers": {
          "X-Forwarded-Proto": "https"
        },
        "host_proxy_headers":[
          "X-Forwarded-Host"
        ],
        "ssl_redirect": true,
        "ssl_host": "ssl.host.domain",
        "sts_seconds": 300,
        "sts_include_subdomains": true,
        "frame_deny": true,
        "referrer_policy": "same-origin",
        "custom_frame_options_value": "ALLOW-FROM https://example.com",
        "content_type_nosniff": true,
        "browser_xss_filter": true,
        "content_security_policy": "default-src 'self';",
        "is_development": false
      }
}
```

See below the different options described in this configuration file.

{{< schema data="security/http.json" >}}


### Restrict connections by host
Use `allowed_hosts` to define the list of hosts that KrakenD accepts requests to. When the list is empty, KrakenD accepts any host.

Each entry must match the `Host` header of the request exactly, ignoring case. The `Host` includes the port only when the client uses a non-default port, so list `api.example.com` for standard HTTPS traffic and `api.example.com:8443` for a custom port. KrakenD rejects requests to any other host with a `500` status code and a `Bad Host` body, and they never reach your backends.

Use `allowed_hosts` whenever a backend relies on `X-Forwarded-Host`, because KrakenD fills that header with the `Host` of the client request (see [non-customizable headers](/docs/endpoints/parameter-forwarding/#non-customizable-headers)).

{{< note title="Proxy headers change which host is checked" type="warning" >}}
When you set `host_proxy_headers` and the request carries one of those headers, `allowed_hosts` checks that header instead of `Host`. KrakenD still forwards the original `Host` to your backends as `X-Forwarded-Host`, so a client can send an allowed value in the proxy header and any value in `Host`. Set `host_proxy_headers` only when KrakenD is not reachable directly and the proxy in front of it always overwrites that header.
{{< /note >}}

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
{{< note title="HPKP is deprecated and has no effect" type="warning" >}}
KrakenD ignores `hpkp_public_key` and does not send the `Public-Key-Pins` header. Browsers no longer support HTTP Public Key Pinning (HPKP), so KrakenD dropped the feature. Remove `hpkp_public_key` from your configuration: `krakend check` still accepts it, so existing configurations keep working, but it does not pin any key.
{{< /note >}}

## OAuth2

KrakenD supports the client credentials grant.

Use this feature if you need to authorize the KrakenD to access your backend services.

See the specific docs for [OAuth2 Client Credentials](/docs/authorization/client-credentials/)
