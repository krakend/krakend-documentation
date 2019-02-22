---
lastmod: 2019-02-22
date: 2019-02-22
toc: true
linktitle: Static responses (stubs)
title: Static Proxy - Adding static/stub data
weight: 70
menu:
  documentation:
    parent: endpoints
---
The **static proxy** can inject static data in your backend responses before returning them.

A typical scenario is **when some of your backends fail**, and you prefer to provide a **stub response** rather than no content at all. Your application can handle better the failure when useful data, even static, is returned.

Another example scenario is to create an endpoint pointing to an unfinished backend where **its functionality is not in production yet**, but your client application needs to go ahead the backend developers and start using the static responses.

There are many other scenarios, and this is why KrakenD offers several **strategies** that you can use to decide whether to inject static data or not.

# Static response strategies
The supported strategies to inject static data are the following:

- `always`: Injects the static data in the response no matter what.
- `success`: Injects the data when all the backends did not fail.
- `complete`: Injects the data when there weren't any errors, all backends gave a response, and the responses completed successfully (no timeouts)
- `errored`: Injects the data when some backend failed
- `incomplete`: When there is no response, or some backend did not return a valid response in time.

Pay attention to the different strategies as they might offer subtle differences. The code associated to these strategies is:

    func staticAlwaysMatch(_ *Response, _ error) bool       { return true }
    func staticIfSuccessMatch(_ *Response, err error) bool  { return err == nil }
    func staticIfErroredMatch(_ *Response, err error) bool  { return err != nil }
    func staticIfCompleteMatch(r *Response, err error) bool { return err == nil && r != nil && r.IsComplete }
    func staticIfIncompleteMatch(r *Response, _ error) bool { return r == nil || !r.IsComplete }

# Adding static responses
To add a static response add under any `endpoint` an `extra_config` entry as follows:

    "extra_config": {
        "github.com/devopsfaith/krakend/proxy": {
            "static": {
                "strategy": "errored",
                "data": {
                    // YOUR STATIC JSON OBJECT GOES HERE
                }
            }
        }
    }

Inside the `strategy` key choose the strategy that fits your use case, and inside `data` you need to add the JSON object as it's returned.

# Static proxy example
The following `/static` endpoint returns `{"foo": 42, "bar": "foobar"}` when the backend returned errors.

    "endpoints": [
            {
                "endpoint": "/static",
                "backend": [
                    {
                        "host": [
                            "http://your.backend"
                        ],
                        "url_pattern": "/foo"
                    }
                ],
                "extra_config": {
                    "github.com/devopsfaith/krakend/proxy": {
                        "static": {
                            "strategy": "errored",
                            "data": {
                                "foo": 42,
                                "bar": "foobar"
                            }
                        }
                    }
                }
            },


