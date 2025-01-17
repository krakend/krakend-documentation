---
lastmod: 2021-11-23
old_version: true
date: 2019-03-21
linktitle: OAuth2 Client credentials
title: Client Credentials Authorization
description: Learn how to implement OAuth 2.0 Client Credentials (2-legged flow) with KrakenD API Gateway to secure your APIs for machine-to-machine communication
weight: 50
#notoc: true
menu:
  community_v2.8:
    parent: "080 Authentication & Authorization"
images:
-   /images/documentation/krakend-oauth2-2-legged.png
meta:
  #since:
  source: https://github.com/krakend/krakend-oauth2-clientcredentials
  namespace:
  - auth/client-credentials
  scope:
  - backend
---

 Through the **OAuth 2.0 Client Credentials Grant**, KrakenD can do a **2-legged OAuth2 flow**, which means that the gateway requests to your authorization server an access token before reaching the backend's protected resources. This token is passed in the "Authorization" header. The token refreshes when needed.

The client credentials **authorize KrakenD, as the client, to access the protected resources**.

Successfully setting the client credentials for a backend means that KrakenD can get the protected content. Still, the endpoint offered to the end-user will be public unless you protect it with [JWT](/docs/v2.8/authorization/jwt-overview/) or another end-user authentication mechanism.

{{< note title="Does this feature generate a new token for each backend request?" type="question" >}}
**No way!** The token will be **automatically refreshed as necessary** (usually when it expires or the server is restarted).
{{< /note >}}


## Configuring OAuth2 Client Credentials
To access a protected resource using client-credentials, add under every `backend` the appropriate `extra_config`.

The namespace used is `"auth/client-credentials"`. Sample configuration below:
```json
{
    "backend": [
        {
            "url_pattern": "/protected-resource",
            "extra_config": {
                "auth/client-credentials": {
                    "client_id": "YOUR-CLIENT-ID",
                    "client_secret": "YOUR-CLIENT-SECRET",
                    "token_url": "https://your.custom.identity.service.tld/token_endpoint",
                    "endpoint_params": {
                        "audience": ["YOUR-AUDIENCE"]
                    }
                }
            }
        }
    ]
}
```
The settings of this component are:

{{< schema version="v2.8" data="auth/client-credentials.json" >}}

## Auth0 integration
The following example demonstrates a complete configuration to fulfill the requirements of [Auth0](https://auth0.com/). It is essentially the same configuration we have shown above, but with some additions, explained after the code:
```json
{
    "endpoint": "/endpoint",
    "backend": [{
        "url_pattern": "/backend",
        "extra_config": {
            "auth/client-credentials": {
                "client_id": "YOUR-CLIENT-ID",
                "client_secret": "YOUR-CLIENT-SECRET",
                "token_url": "https://custom.auth0.tld/token_endpoint",
                "endpoint_params": {
                    "client_id": ["YOUR-CLIENT-ID"],
                    "client_secret": ["YOUR-CLIENT-SECRET"],
                    "audience": ["YOUR-AUDIENCE"]
                }
            },
            "modifier/martian": {
                "fifo.Group": {
                    "scope": ["request", "response"],
                    "aggregateErrors": false,
                    "modifiers": [
                        {
                            "header.Modifier": {
                                "scope": ["request"],
                                "name" : "Accept",
                                "value" : "application/json"
                            }
                        }
                    ]
                }
            }
        }
    }]
}
```

The code above works with Auth0. The difference with the basic example is the way both the id and the secret are passed as `endpoint_params`, as auth0 ignores the auth header and expects the credentials sent as JSON data or form body.
