---
lastmod: 2022-08-29
date: 2019-01-24
linktitle: Transform requests and responses
title: Modify requests and responses with Martian
weight: 85
aliases: ["/docs/endpoints/martian"]
menu:
  community_current:
    parent: "050 Backends Configuration"
meta:
  since: 0.7
  source: https://github.com/krakendio/krakend-martian
  namespace:
  - modifier/martian
  scope:
  - backend
---
The [krakend-martian](https://github.com/krakendio/krakend-martian) component allows you to **transform requests and responses** through a simple DSL definition in the configuration file. Martian works perfectly in combination with [CEL verifications](/docs/endpoints/common-expression-language-cel/).

Use Martian when you want to intercept the end-user's request and make modifications before passing the content to the backends. Also, the other way around, transform the backends response before passing it to the user.

Martian is mighty and gives you endless possibilities to control what is going in and out of the gateway. Some **examples of typical Martian scenarios** are:

- Set a new cookie during gateway processing
- Add, remove or change specific headers
- Query string additions before making the backend request (e.g., set an API key)

There are four different types of interactions you can do with Martian:

- **Modifiers**: Change the state of a request or a response. For instance, you want to add a custom header in the request before sending it to a backend.
- **Filters**: Add a condition to execute a contained Modifier
- **Groups**: Bundle multiple operations to execute in the order specified in the group
- **Verifiers**: Track network traffic against expectations

## Transforming requests and responses

Add martian modifiers in your configuration under the `extra_config` of any `backend` using the namespace `modifier/martian`.

Your configuration has to look as follows:

```json
{
  "backend": [
    {
      "url_pattern": "/foo/{var}",
      "extra_config": {
        "modifier/martian": {
          // modifier configuration here
        }
      }
    }
  ]
}
```

See the possibilities and examples below.

{{< note title="A note on client headers" >}}
When **client headers** are needed, remember to add them under [`input_headers`](/docs/endpoints/parameter-forwarding/#headers-forwarding) as KrakenD does not forward headers to the backends unless declared in the list.
{{< /note >}}


### Modifier configuration

In the examples below, you'll find that all modifiers have a configuration key named `scope`. The scope indicates when to apply the modifier. It can be an array containing `request`, `response`, or both. The rest of the keys in every modifier depends on the modifier itself.


## Transform headers
The `header.Modifier` injects a header with a specific value. For instance, the following configuration adds a header `X-Martian` both in the request and the response.

```json
{
  "backend": [
    {
      "url_pattern": "/foo/{var}",
      "extra_config": {
        "modifier/martian": {
          "header.Modifier": {
            "scope": [
              "request",
              "response"
            ],
            "name": "X-Martian",
            "value": "true"
          }
        }
      }
    }
  ]
}
```
Notice that even there is a response modifier, **the client won't see these headers**, as KrakenD's policy is to not return the headers to the client by default. KrakenD will see the new `X-Martian` header at the merge stage, but the client won't. The backend will see the header in its request though.

## Modify the body
Through the `body.Modifier`, you can modify the body of the request and the response. You must encode in `base64` the content of the `body`.

The following modifier sets the body of the request and the response to `{"msg":"you rock!"}`. Notice that the `body` field is `base64` encoded (e.g., `echo "content" | base64 -w0`).

```json
{
  "backend": [
    {
      "url_pattern": "/foo/{var}",
      "extra_config": {
        "modifier/martian": {
          "body.Modifier": {
            "scope": [
              "request",
              "response"
            ],
            "body": "eyJtc2ciOiJ5b3Ugcm9jayEifQ=="
          }
        }
      }
    }
  ]
}
```

The [Flexible Configuration](/docs/configuration/flexible-config/) has a `b64enc` function that will allow you to have an easier to read configuration. For instance (notice the backticks as delimiters):

```go-text-template
"body": "{{- `{"msg":"you rock!"}` | b64enc -}}"
```
Or from an external file:
```go-text-template
"body": "{{- include "external_file.txt" | b64enc -}}"
```

## Transform the URL
The `url.Modifier` allows you to change settings in the URL. For instance:

```json
{
    "extra_config": {
        "modifier/martian": {
            "url.Modifier": {
              "scope": ["request"],
              "scheme": "https",
              "host": "www.google.com",
              "path": "/proxy",
              "query": "testing=true"
            }
        }
    }
}
```

## Copying headers
Although not widely used, the `header.Copy` lets you duplicate a header using another name. Remember that any header you want to access here it must be included in the `input_headers` list.

```json
{
    "extra_config": {
        "modifier/martian": {
            "header.Copy": {
                "scope": ["request", "response"],
                "from": "Original-Header",
                "to": "Copy-Header"
            }
        }
    }
}
```


## Apply multiple modifiers consecutively
All the examples above perform a single modification in the request or the response. However, the `fifo.Group` allows you to create a list of modifiers that execute consecutively. The group is needed when using more than one modifier and encapsulates all the following actions to perform in the `modifiers` array. You can use the FIFO group even when there is only one modifier in the list.


Example of usage (modify the body, and set a header):

```json
{
  "backend": [
    {
      "url_pattern": "/foo/{var}",
      "extra_config": {
        "modifier/martian": {
          "fifo.Group": {
            "scope": [
              "request",
              "response"
            ],
            "aggregateErrors": true,
            "modifiers": [
              {
                "body.Modifier": {
                  "scope": [
                    "request"
                  ],
                  "body": "eyJtc2ciOiJ5b3Ugcm9jayEifQ=="
                }
              },
              {
                "header.Modifier": {
                  "scope": [
                    "request",
                    "response"
                  ],
                  "name": "X-Martian",
                  "value": "true"
                }
              }
            ]
          }
        }
      }
    }
  ]
}
```

## Connecting to Basic Auth (user/pass) backends
Sometimes your backends are protected, and you need KrakenD to provide a user and password to connect. The basic authentication requires you to provide a header with the form `Authorization: Basic <credentials>`. The credentials are the concatenation of the username and password using a colon `:` in base64.

For instance, if your username is `user` and your password `pa55w0rd`, you should generate the base64 as follows:

{{< terminal title="Term" >}}
echo -n "user:pa55w0rd" | base64
dXNlcjpwYTU1dzByZA==
{{< /terminal >}}

When using `echo`, make sure to add the `-n` option to avoid the final line break from being encoded. You can test if the connection succeeds now with:

{{< terminal title="Term" >}}
curl -i https://yourapi --header 'Authorization: Basic dXNlcjpwYTU1dzByZA=='
{{< /terminal >}}


If the connection works, it means that your credentials are correct, and you can add the resulting base64 string `dXNlcjpwYTU1dzByZA==` to the Martian modifier right before connecting to your `backend`:

```json
{
    "url_pattern": "/protected",
    "extra_config": {
        "modifier/martian": {
            "header.Modifier": {
              "scope": ["request"],
              "name": "Authorization",
              "value": "Basic dXNlcjpwYTU1dzByZA=="
            }
        }
    }
}
```

With the configuration above, whenever a request is made to the backend, the `Authorization` header is added automatically.

For more complex examples of authentication using Martian, see the example [Adding automatic API authentication](/blog/website-development-as-a-sysadmin/)

## All Martian modifiers, verifiers, and filters
The Martian library comes with [+25 modifiers](https://github.com/google/martian) you can use. We are not listing all the options in the documentation. Instead, we provided the modifiers that are key when using Martian.

For the complete list of modifiers and usage, see [Google's Martian repository](https://github.com/google/martian). These are the packages included in KrakenD-CE:

- [github.com/google/martian/body](https://github.com/google/martian/tree/master/body)
- [github.com/google/martian/cookie](https://github.com/google/martian/tree/master/cookie)
- [github.com/google/martian/fifo](https://github.com/google/martian/tree/master/fifo)
- [github.com/google/martian/header](https://github.com/google/martian/tree/master/header)
- [github.com/google/martian/martianurl](https://github.com/google/martian/tree/master/martianurl)
- [github.com/google/martian/port](https://github.com/google/martian/tree/master/port)
- [github.com/google/martian/priority](https://github.com/google/martian/tree/master/priority)
- [github.com/google/martian/querystring](https://github.com/google/martian/tree/master/querystring)
- [github.com/google/martian/stash](https://github.com/google/martian/tree/master/stash)
- [github.com/google/martian/status](https://github.com/google/martian/tree/master/status)

## Building new modifiers
Sometimes you want to create a new modifier that covers your specific use case and does some other dynamic operations. Creating more modifiers is a straightforward process and only requires `make` the gateway once you have it coded.

Nothing better than an example to demonstrate how to create a new modifier. Our SRE Director (who wasn't familiar with Go) went through the process of creating a new Modifier that would authenticate automatically against the Marvel API, adding an API Key, a timestamp, and a calculated hash.

Read it here (contains the source code): [Martian example: Adding automatic API authentication](/blog/website-development-as-a-sysadmin/)
