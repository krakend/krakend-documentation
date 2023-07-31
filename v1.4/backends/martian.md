---
lastmod: 2021-05-02
old_version: true
date: 2019-01-24
linktitle: Transform requests and responses
title: Modify requests and responses with Martian
weight: 85
menu:
  community_v1.4:
    parent: "050 Backends Configuration"
meta:
  since: 0.7
  source: https://github.com/krakend/krakend-martian
  namespace:
  - github.com/devopsfaith/krakend-martian
  scope:
  - backend
---
The [krakend-martian](https://github.com/krakend/krakend-martian) component allows you to **transform requests and responses** through a simple DSL definition in the configuration file. Martian works perfectly in combination with [CEL verifications](/docs/v1.4/endpoints/common-expression-language-cel/).

Use Martian when you want to intercept the request of the end-user and make modifications before passing the content to the backends. Also, the other way around, transform the backends response before passing it to the user.

Martian is mighty and gives you endless possibilities to control what is going in and out the gateway. Some **examples of typical Martian scenarios** are:

- Set a new cookie during gateway processing
- Add, remove or change specific headers
- Query string additions before making the backend request (e.g., set an API key)

There are four different types of interactions you can do with Martian:

- **Modifiers**: Change the state of a request or a response. For instance, you want to add a custom header in the request before sending it to a backend.
- **Filters**: Add a condition to execute a contained Modifier
- **Groups**: Bundle multiple operations to execute in the order specified in the group
- **Verifiers**: Track network traffic against expectations

## Transforming requests and responses

Add martian modifiers in your configuration under the `extra_config` of any `backend` using the namespace `github.com/devopsfaith/krakend-martian`.

Your configuration has to look as follows:

    "extra_config": {
        "github.com/devopsfaith/krakend-martian": {
            // modifier configuration here
        }
    }

See the possibilities and examples below.

{{< note title="A note on client headers" >}}
When **client headers** are needed, remember to add them under [`headers_to_pass`](/docs/v1.4/endpoints/parameter-forwarding/#headers-forwarding) as KrakenD does not forward headers to the backends unless declared in the list.
{{< /note >}}


### Modifier configuration

In the examples below, you'll find that all modifiers have a configuration key named `scope`. The scope indicates when to apply the modifier, it can be an array containing `request`, `response`, or both. The rest of the keys in every modifier depends on the modifier itself.


## Transform headers
The `header.Modifier` injects a header with a specific value. For instance, the following configuration adds a header `X-Martian` both in the request and the response.

    "extra_config": {
        "github.com/devopsfaith/krakend-martian": {
            "header.Modifier": {
              "scope": ["request", "response"],
              "name": "X-Martian",
              "value": "true"
            }
        }
    }

## Modify the body
Through the `body.Modifier` you can modify the body of the request and the response. The content of the `body` must be encoded in `base64`.

The following modifier sets the body of the request and the response to `{"msg":"you rock!"}`. Notice that the `body` field is `base64` encoded.

    "extra_config": {
        "github.com/devopsfaith/krakend-martian":
          {
              "body.Modifier": {
                  "scope": ["request","response"],
                  "body": "eyJtc2ciOiJ5b3Ugcm9jayEifQ=="
              }
          }
    }


## Transform the URL
The `url.Modifier` allows you to change settings in the URL. For instance:

    "extra_config": {
        "github.com/devopsfaith/krakend-martian": {
            "url.Modifier": {
              "scope": ["request"],
              "scheme": "https",
              "host": "www.google.com",
              "path": "/proxy",
              "query": "testing=true"
            }
        }
    }

## Copying headers
Although not widely used, the `header.Copy` lets you duplicate a header using another name.

    {
      extra_config": {
        "github.com/devopsfaith/krakend-martian": {
          "header.Copy": {
            "scope": ["request", "response"],
            "from": "Original-Header",
            "to": "Copy-Header"
            }
        }
      }
    }

## Apply multiple modifiers consecutively
All the examples above perform a single modification in the request or the response. However, the `fifo.Group` allows you to create a list of modifiers that execute consecutively. The group is needed when using more than one modifier and encapsulates all the following actions to perform in the `modifiers` array. You can use the FIFO group even when there is only one modifier in the list.


Example of usage (modify the body, and set a header):

    "extra_config": {
        "github.com/devopsfaith/krakend-martian": {
            "fifo.Group": {
                "scope": ["request", "response"],
                "aggregateErrors": true,
                "modifiers": [
                    {
                        "body.Modifier": {
                            "scope": ["request"],
                            "body": "eyJtc2ciOiJ5b3Ugcm9jayEifQ=="
                        }
                    },
                    {
                      "header.Modifier": {
                        "scope": ["request", "response"],
                        "name": "X-Martian",
                        "value": "true"
                      }
                    }
                ]
            }
        }
    }

## All Martian modifiers, verifiers and filters
The Martian library comes with [+25 modifiers](https://github.com/google/martian) you can use, we are not listing all the options in the documentation. Instead, we provided the modifiers that are key when using Martian.

For the complete list of modifiers and usage see [Google's Martian repository](https://github.com/google/martian). These are the packages included in KrakenD-CE:

- [github.com/google/martian/body](https://github.com/google/martian/tree/master/body)
- [github.com/google/martian/cookie](https://github.com/google/martian/tree/master/cookie)
- [github.com/google/martian/fifo](https://github.com/google/martian/tree/master/fifo)
- [github.com/google/martian/header](https://github.com/google/martian/tree/master/header)
- [github.com/google/martian/martianurl](https://github.com/google/martian/tree/master/martianurl)
- [github.com/google/martian/port](https://github.com/google/martian/tree/master/port)
- [github.com/google/martian/priority](https://github.com/google/martian/tree/master/priority)
- [github.com/google/martian/stash](https://github.com/google/martian/tree/master/stash)
- [github.com/google/martian/status](https://github.com/google/martian/tree/master/status)

## Building new modifiers
There will be times when you want to create a new modifier that covers your specific use case and does some other dynamic operations. Creating more modifiers is a straightforward process and only requires to `make` the gateway once you have it coded.

Nothing better than an example to demonstrate how to create a new modifier. Our SRE Director (who wasn't familiar with Go) went through the process of creating a new Modifier that would authenticate automatically against the Marvel API adding an API Key, a timestamp and a calculated hash.

Read it here (contains the source code): [Martian example: Adding automatic API authentication](/blog/website-development-as-a-sysadmin/)
