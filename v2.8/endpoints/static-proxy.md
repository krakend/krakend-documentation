---
lastmod: 2024-07-03
old_version: true
date: 2019-02-22
linktitle: Static responses on failures (stubs)
title: Static Proxy Endpoints
description: Learn how to configure static proxy endpoints in KrakenD API Gateway to return stubs and static data on certain events
weight: 440
menu:
  community_v2.8:
    parent: "060 Request and Response Manipulation"
meta:
  noop_incompatible: true
  since: 0.5
  source: https://github.com/luraproject/lura
  namespace:
  - proxy
  scope:
  - endpoint
  - async_agent
---
The **static proxy** aids in decorating the final response with static data. It helps deal with incomplete and degraded responses or add more content to good responses. When enabled, it injects the static `data`** into the final response when a backend's behavior falls within the selected strategy. The `data` is **injected** replacing any colliding keys and merging with any existing data from the backend responses.

{{< note title="Key precedence" type="info" >}}
The static proxy `data` is a final decorator and wins every fight. Any keys you add under the `data` of the static proxy, when triggered, take precedence over any existing response. If your static data uses the **same keys** you use in your backends, those colliding with the upstream will be replaced.
{{< /note >}}

A typical scenario for using the static proxy is **when some backend fails** and the endpoint becomes incomplete, but you prefer to provide a **stub/mock response** to enhance the functionality. When your application cannot handle well the degraded response, the static data comes in handy.

Another example scenario is to **connect to an unfinished or unexisting backend**. While the backend development is not still there, the client application can have mock data that will become real data as soon as the backend starts responding.

There are many other scenarios, and this is why KrakenD offers several **strategies** for deciding whether to inject static data. In any case, remember that the primary goal of this feature is to support **corner cases** related to clients not ready to deal with gracefully degraded responses.

## Static response strategies
The supported strategies to inject static data are the following:

- `always`: Injects the static data in the response no matter what.
- `success`: Injects the data when there were no errors. The errors can come from the backend, but also from CEL evaluations, Security Policies and other components that add conditions.

- `complete`: Injects the data when there weren't any errors, all backends gave a response, and the responses merged successfully
- `errored`: Injects the data when some backend fails, returning an explicit error.
- `incomplete`: When some backend did not reach the merge operation (timeout or another reason). This strategy does not support `no-op` as there is no evaluation of the response.

Pay attention to the different strategies as they might offer **subtle differences**. The following table shows when each of the strategies will return the content inside its `data`:


| Strategy      | Condition to trigger |
| ------------- | ------------- |
| `always` | `true` |
| `success` | `err == nil` |
| `complete` | `err == nil and Response != nil and Response.IsComplete` |
| `errored` | `err != nil` |
| `incomplete` | `Response == nil or !Response.IsComplete` |

As you can see, a strategy `success` or `complete` are essentially the same, but the latter is more demanding. If you doubt between `errored` and `incomplete`, choose `errored`, because an incomplete response is surely provoked by some error.

## Handling collisions
As we said above, **the static proxy is processed after all the backend merging** has occurred, meaning that if your static data has keys colliding with the existing responses, these are overwritten. The static proxy is the last execution, taking precedence over any existing response.

When an endpoint aggregates data from multiple sources, if a `group` for each backend is not used, then all the responses are merged straight into the root. The static data makes the merge in the root as well, so be cautious when setting the content of `data`, to make sure you are not replacing valuable information.

A piece of advice is that you add a new key under `data` to avoid undesired replacements.

## Adding static responses
To add a static response, add under any `endpoint` an `extra_config` entry as follows:

```json
{
    "endpoint": "/static",
    "extra_config": {
        "proxy": {
            "static": {
                "strategy": "errored",
                "data": {
                    "YOUR STATIC JSON OBJECT": "GOES HERE"
                }
            }
        }
    }
}
```

Inside the `strategy` key choose the strategy that fits your use case (one of `always`, `success`, `complete`, `errored`or `incomplete`), and inside `data` you need to add the JSON object as it's returned.

## Static proxy example
The following configuration declares two endpoints that will fail that you can test locally (copy and paste without replacements):


```json
{
    "version": 3,
    "endpoints": [
        {
            "endpoint": "/static/errored",
            "backend": [
                {
                    "host": [
                        "http://example.com"
                    ],
                    "url_pattern": "/foo",
                    "group": "foo"
                },
                {
                    "host": [
                        "http://example.com"
                    ],
                    "url_pattern": "/bar",
                    "group": "bar"
                }
            ],
            "extra_config": {
                "proxy": {
                    "static": {
                        "strategy": "errored",
                        "data": {
                            "errors_happened": {
                                "foo": 42,
                                "bar": "default bar"
                            }
                        }
                    }
                }
            }
        },
        {
            "endpoint": "/static/incomplete",
            "backend": [
                {
                    "host": [
                        "http://example.com"
                    ],
                    "url_pattern": "/foo",
                    "group": "foo"
                },
                {
                    "host": [
                        "http://localhost:8080"
                    ],
                    "url_pattern": "/__health",
                    "group": "bar"
                }
            ],
            "extra_config": {
                "proxy": {
                    "static": {
                        "strategy": "incomplete",
                        "data": {
                            "defaults": {
                                "foo": 42,
                                "bar": "default bar"
                            }
                        }
                    }
                }
            }
        }
    ]
}
```


- `/static/errored` tries to connect to two unexisting backends which will make the whole endpoint to fail at getting data, and uses the `errored` strategy in the proxy. When this happens, we return a static object that could contain some data to allow the client render something. In this case we are returning:
    ```json
    {
        "errors_happened": {
            "foo": 42,
            "bar": "default bar"
        }
    }
    ```
- `/static/incomplete` connects to a failing backend, but the other works, so we get some information. We return an object with default values the client could use to render the missing content:
    ```json
    {
        "defaults": {
            "foo": 42,
            "bar": "default bar"
        }
    }
    ```

At a practical level, the two strategies are almost the same, only that `incomplete` won't work with `no-op`.

Notice two things in the example endpoints:

1. Each backend uses a `group`, so when they work correctly, responses are inside a key "foo" or "bar". Using this strategy, if "foo" and "bar" contain duplicate keys in the response, there is no overwrite problem.
2. The strategies do not know which backend failed or had errors, so having a different key like `defaults` or `errors_happened` allows you to have both the default data and the partial responses in the final response.

And remember: the static proxy is the last piece of software evaluated, so it merges with the responses (if any) and can overwrite good data, so keeping different keys is advisable.
