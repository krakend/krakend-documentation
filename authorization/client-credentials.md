---
lastmod: 2021-11-23
date: 2019-03-21
linktitle: OAuth2 Client credentials
title: OAuth 2.0 Client Credentials (2-legged flow)
weight: 50
#notoc: true
menu:
  community_current:
    parent: "060 Authentication & Authorization"
images:
-   /images/documentation/krakend-oauth2-2-legged.png
meta:
  #since: 
  source: https://github.com/devopsfaith/krakend-oauth2-clientcredentials
  namespace:
  - auth/validator
  scope:
  - endpoint
---

Through the **OAuth 2.0 Client Credentials Grant**, KrakenD can request to your authorization server an access token to reach protected resources. This is frequently described as the **2-legged OAuth2 flow**.

The client credentials **authorize KrakenD, as the client, to access the protected resources**. Do not confuse this with authorizing an end-user (see [JWT](/docs/authorization/jwt-overview/) instead).

Successfully setting the client credentials for a backend means that KrakenD can get the protected content, but the endpoint offered to the end-user is going to be public unless you protect it with JWT.

## Configuring OAuth2 Client Credentials
To access a protected resource using client-credentials add under every `backend` the appropriate `extra_config`.

The namespace used is `"auth/client-credentials"`. Sample configuration below:
{{< highlight json >}}
{
    "endpoint": "/endpoint",
    "backend": [{
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
    }]
}
{{< /highlight >}}
The settings of this component are:

- `client_id` *string*: The Client ID provided to the Auth server
- `client_secret` *string*: The secret string provided to the Auth server
- `token_url` *string*: The endpoint URL where the negotiation of the token happens
- `scopes`: *string,optional* A comma separated list of scopes needed, e.g.: `scopeA,scopeB`
- `endpoint_params` *list,optional*: Any additional parameters that you want to include **in the payload** when requesting the token. For instance, it is frequent to add the `audience` request parameter that denotes the target API for which the token should be issued.

{{< note title="Does this feature generate a new token for each backend request?" type="question" >}}
No way! The token will be **automatically refreshed as necessary** (usually when it expires or the server is restarted).
{{< /note >}}

## Auth0 integration
The following example demonstrates a complete configuration to fulfill the requirements of [Auth0](https://auth0.com/). It is essentially the same configuration we have shown above, but with some additions, explained after the code:
{{< highlight json >}}
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
{{< /highlight >}}

The code above works with Auth0. The difference with the basic example is the way both the id and the secret are passed as `endpoint_params`, as auth0 ignores the auth header and expects the credentials sent as JSON data or form body.
