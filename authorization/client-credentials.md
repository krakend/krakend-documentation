---
lastmod: 2019-03-21
date: 2019-03-21
linktitle: OAuth2 Client credentials
title: OAuth 2.0 Client Credentials
source: https://github.com/devopsfaith/krakend-oauth2-clientcredentials
weight: 50
#notoc: true
menu:
  community_current:
    parent: "060 Authentication & Authorization"
---

Through the **OAuth 2.0 Client Credentials Grant** KrakenD can request to your authorization server an access token to reach protected resources.

The client credentials **authorize KrakenD, as the client, to access the protected resources**. Do not confuse this with authorizing an end-user (see [JWT](/docs/authorization/jwt-overview/) instead).

Successfully setting the client credentials for a backend means that KrakenD can get the protected content, but the endpoint offered to the end-user is going to be public unless you protect it with JWT.

## Configuring OAuth2 Client Credentials
To access a protected resource using client-credentials add under every `backend` the appropriate `extra_config`.

The namespace used is `"github.com/devopsfaith/krakend-oauth2-clientcredentials"`. Sample configuration below:

    {
      "endpoint": "/endpoint",
      "backend": [
          {
              "url_pattern": "/protected-resource",
              "extra_config": {
                  "github.com/devopsfaith/krakend-oauth2-clientcredentials": {
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

The settings of this component are:

- `client_id` *string*: The Client ID provided to the Auth server
- `client_secret` *string*: The secret string provided to the Auth server
- `token_url` *string*: The endpoint URL where the negotiation of the token happens
- `scopes`: *string,optional* A comma separated list of scopes needed, e.g.: `scopeA,scopeB`
- `endpoint_params` *list,optional*: Any additional parameters that you want to include **in the payload** when requesting the token. For instance, it is frequent to add the `audience` request parameter that denotes the target API for which the token should be issued.


## Auth0 integration
The following example demonstrates a complete configuration to fulfill the requirements of [Auth0](https://auth0.com/). It is essentially the same configuration we have shown above, but with some additions, explained after the code:

    {
        "endpoint": "/endpoint",
        "backend": [
            {
                "url_pattern": "/backend",
                "extra_config": {
                    "github.com/devopsfaith/krakend-oauth2-clientcredentials": {
                        "client_id": "YOUR-CLIENT-ID",
                        "client_secret": "YOUR-CLIENT-SECRET",
                        "token_url": "https://custom.auth0.tld/token_endpoint",
                        "endpoint_params": {
                            "client_id": ["YOUR-CLIENT-ID"],
                            "client_secret": ["YOUR-CLIENT-SECRET"],
                            "audience": ["YOUR-AUDIENCE"]
                        }
                    },
                    "github.com/devopsfaith/krakend-martian": {
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
            }
        ]
    }

The code above works with Auth0. The difference with the basic example is the way both the id and the secret are passed as `endpoint_params`, as auth0 ignores the auth header and expects the credentials sent as JSON data or form body.
