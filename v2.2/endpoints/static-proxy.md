---
lastmod: 2019-02-22
old_version: true
date: 2019-02-22
linktitle: Static responses (stubs)
title: Static Proxy - Adding static/stub data
weight: 70
menu:
  community_v2.2:
    parent: "040 Endpoint Configuration"
meta:
  noop_incompatible: true
  since: v0.5
  source: https://github.com/luraproject/lura
  namespace:
  - proxy
  scope:
  - endpoint
  - async_agent
---
The **static proxy** is an aid to clients dealing with incomplete and other types of degraded responses. When enabled, the static proxy **injects static data** in the final response when the behavior of a backend falls in the selected **strategy**.

A typical scenario is **when some backend fails** and the endpoint becomes incomplete, but you prefer to provide a **stub response** for that part instead. When your application cannot handle well the degraded response, the static data comes handy.

Another example scenario is to create an endpoint pointing to an unfinished backend where **its functionality is not in production yet**, but your client application needs to go ahead the backend developers and start using the static responses.

There are many other scenarios, and this is why KrakenD offers several **strategies** that you can use to decide whether to inject static data or not. In any case, remember that the primary goal of this feature is to support corner-cases related to clients not ready to deal with gracefully degraded responses.

## Static response strategies
The supported strategies to inject static data are the following:

- `always`: Injects the static data in the response no matter what.
- `success`: Injects the data when all the backends did not fail.
- `complete`: Injects the data when there weren't any errors, all backends gave a response, and the responses merged successfully
- `errored`: Injects the data when some backend failed, returning an explicit error.
- `incomplete`: When some backend did not reach the merge operation (timeout or another reason).

Pay attention to the different strategies as they might offer subtle differences. The code associated to these strategies is:

```go
func staticAlwaysMatch(_ *Response, _ error) bool       { return true }
func staticIfSuccessMatch(_ *Response, err error) bool  { return err == nil }
func staticIfErroredMatch(_ *Response, err error) bool  { return err != nil }
func staticIfCompleteMatch(r *Response, err error) bool { return err == nil && r != nil && r.IsComplete }
func staticIfIncompleteMatch(r *Response, _ error) bool { return r == nil || !r.IsComplete }
```



## Handling collisions
The static proxy is processed **after** all the backend merging has occurred, meaning that if your static data has keys that are colliding with the existing responses, these are overwritten.

The static data always has a priority as it's the last computed part. When an endpoint aggregates data from multiple sources, if a `group` for each backend is not used, then all the responses are merged straight into the root. The static data makes the merge in the root as well, so be cautious when setting the content of `data`, to make sure you are not replacing valuable information.

## Adding static responses
To add a static response add under any `endpoint` an `extra_config` entry as follows:

```json
{
    "endpoint": "/static",
    "extra_config": {
        "proxy": {
            "static": {
                "strategy": "errored",
                "data": {
                    YOUR STATIC JSON OBJECT GOES HERE
                }
            }
        }
    }
}
```

Inside the `strategy` key choose the strategy that fits your use case (one of `always`, `success`, `complete`, `errored`or `incomplete`), and inside `data` you need to add the JSON object as it's returned.

## Static proxy example
The following `/static` endpoint returns `{"errored": {"foo": 42, "bar": "foobar"} }` when the backend returned errors.

Notice two things in the example trying to avoid collisions.  First, each backend uses a `group`, so when the backend works correctly, its response is inside a key "foo" or "bar". Using this strategy if "foo" and "bar" use the same keys there is no problem.

Secondly, when one of the 2 backends fail, it creates a new group "oh-snap" (see `data`).

```json
{
    "endpoints": [
        {
            "endpoint": "/static",
            "backend": [
                {
                    "host": ["http://your.backend"],
                    "url_pattern": "/foo",
                    "group": "foo"
                },
                {
                    "host": ["http://your.backend"],
                    "url_pattern": "/bar",
                    "group": "bar"
                }
            ],
            "extra_config": {
                "proxy": {
                    "static": {
                        "strategy": "errored",
                        "data": {
                            "oh-snap": {
                                "id": 42,
                                "bar": "foobar"
                            }
                        }
                    }
                }
            }
        }
    ]
}
```
