---
lastmod: 2018-10-29
date: 2018-10-29
linktitle: Traffic shadowing/mirroring
title: Traffic shadowing or mirroring
weight: 60
since: 0.9
notoc: true
menu:
  community_current:
    parent: "050 Backends Configuration"
meta:
  noop_incompatible: true
---
There are times when you have been working in a new version of your microservice, a complete refactor, a dangerous change, or any other valuable change that needs being careful, and it's too risky to put it live as there might be issues that impact your end users.

The **traffic shadowing** or **traffic mirroring** functionality allows you to **test new backends in production** by sending them copies of the traffic but **ignore their responses**.

When you add a backend to any of your endpoints as a **shadow backend**, KrakenD continues to send the requests to all the backends as usual, but the responses from the ones marked as *shadow* are ignored and never returned or merged in the response.

Mirroring the traffic to your microservices allows you to test your new backend from the interesting perspective of seeing behavior in production. For instance, you could:

- Test the application errors by examining its logs
- Test the performance of the application
- Stress a new server
- Retrieve any other interesting data that you can only see when you have something running in production.

To define a backend as a *shadow backend* you only need to add the flag as follows:

```json
{
    "extra_config": {
        "proxy": {
            "shadow": true
        }
    }
}
```

With this change, the backend containing the flag enters into production, but KrakenD ignores its responses.

## Traffic shadowing example
The following example shows a backend that is changing from `v1` to `v2`, but we are still unsure of the effects of doing this change in production, so we want to send a copy of the requests to `v2` in the first place, but keep the end users receiving the responses from `v1` only:

```json
{
    "endpoint": "/user/{id}",
    "timeout": "150ms",
    "backend": [
        {
            "host": [ "http://my.api.com" ],
            "url_pattern": "/v1/user/{id}"
        },
        {
            "host": [ "http://my.api.com" ],
            "url_pattern": "/v2/user/{id}",
            "extra_config": {
                "proxy": {
                    "shadow": true
                }
            }
        }
    ]
}
```
