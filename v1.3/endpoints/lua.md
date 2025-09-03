---
lastmod: 2021-05-02
old_version: true
date: 2019-09-15
linktitle:  Lua scripting
title: Transformations using Lua scripting
weight: 90
menu:
  community_v1.3:
    parent: "040 Endpoint Configuration"
meta:
  since: v1.0
  source: https://github.com/krakend/krakend-lua
  namespace:
  - "github.com/devopsfaith/krakend-lua/proxy"
  - "github.com/devopsfaith/krakend-lua/router"
  - "github.com/devopsfaith/krakend-lua/proxy/backend"
  scope:
  - endpoint
  - backend
---

Scripting with Lua is an additional choice to extend your business logic, and is compatible with the rest of options such as [CEL](/docs/v1.3/endpoints/common-expression-language-cel/), [Martian](/docs/v1.3/backends/martian/), or other Go plugins and middlewares.

If you are more familiar with Lua than Go, this module can help you solve exceptional cases that need solution using a little bit of scripting. The introduction of Lua scripts in your Gateway does not require to recompile KrakenD, but unlike Go, Lua scripts are interpreted in real-time.

For performance-first users, a Go plugin delivers much better results than a Lua script.

## Configuration

KrakenD looks for the lua scripts in the root folder where KrakenD is running. You need to specify in the configuration which lua scripts are going to be loaded in Krakend, as well as several options. The `extra_config` can be set at `endpoint` level or `backend` level.

    "extra_config": {
          "github.com/devopsfaith/krakend-lua/proxy": {
            "sources": ["file1.lua"],
            "md5": {
              "file1.lua": "49ae50f58e35f4821ad4550e1a4d1de0"
            },
            "pre": "lua code to execute for pre",
            "post": "lua code to execute for post",
            "live": false,
            "allow_open_libs": false,
            "skip_next": true
          }
    }

- `sources`: An array with all the files that will be processed
- `md5`: (optional) The md5sum of each file that must match the one found in the disk. Used to make sure that the file has not been modified by a 3rd party.
- `pre` and `post` contain the code to start the execution in every step. `post` is only available in the `backend` section.
- `live`: Live reload of the script in every execution
- `allow_open_libs`: The regular lua libraries are not open by default, as an efficiency point. But if you need to use the lua libraries (for file io for example), then set this to true.  If not present, default value is false.
- `skip_next`: only to be set when in a `backend` section, skips the query to the next backend.

{{< note title="A note on client headers" >}}
When **client headers** are needed, remember to add them under [`headers_to_pass`](/docs/v1.3/endpoints/parameter-forwarding/#headers-forwarding) as KrakenD does not forward headers to the backends unless declared in the list.
{{< /note >}}

## Namespaces (component name)

There are three namespaces that are used for the lua component.

Under the `endpoint` section use the namespaces (these are described in the next section):

- `"github.com/devopsfaith/krakend-lua/proxy"`
- `"github.com/devopsfaith/krakend-lua/router"`

Under the `backend` use the name space:

- `"github.com/devopsfaith/krakend-lua/proxy/backend"`

## Supported Lua types (cheatsheet)

When running Lua scripts on KrakenD, there are two different types you can use in their coding. Depending on the place of the pipe you want to place the script you can use a `proxy` type or a `router` type:

    End User <--[router]--> KrakenD <--[proxy]--> Services

These two types are described as follows:

- Router: The router layer is what happens between the end-user and KrakenD
- Proxy: The proxy layer is between KrakenD and your services

### `proxy` type

Use this type when you need to intercept requests and responses between KrakenD and your services.

#### Request

Scripts that need to modify a request that KrakenD is about to do against the backend services.

*   `load` (_Static_)
*   `method` (_Dynamic_)
*   `path` (_Dynamic_)
*   `query` (_Dynamic_)
*   `url` (_Dynamic_)
*   `params` (_Dynamic_)
*   `headers` (_Dynamic_)
*   `body` (_Dynamic_)

Example: Access the request getter with `req:url()` and the setter with `req:url("foo")`.

#### Response

Scripts that need to modify a request that KrakenD is about to get from the backend services.

*   `load` (_Static_)
*   `isComplete` (_Dynamic_)
*   `statusCode` (_Dynamic_)
*   `data` (_Dynamic_)
*   `headers` (_Dynamic_)
*   `body` (_Dynamic_)

### `router` type

Use this type when you need to script the router layer, traffic between end-users and KrakenD.

#### ctx

*   `load` (_Static_)
*   `method` (_Dynamic_)
*   `query` (_Dynamic_)
*   `url` (_Dynamic_)
*   `params` (_Dynamic_)
*   `headers` (_Dynamic_)
*   `body` (_Dynamic_)
*   `host` (_Dynamic_)

## Additional helpers (cheatsheet)

The following helpers are available in your scripts:

### `table`

*   `get` (_Dynamic_)
*   `set` (_Dynamic_)
*   `len` (_Dynamic_)

### `list`

*   `get` (_Dynamic_)
*   `set` (_Dynamic_)
*   `len` (_Dynamic_)

### `http_response`

*   `new` (_Static_)
*   `statusCode` (_Dynamic_)
*   `headers` (_Dynamic_)
*   `body` (_Dynamic_)

### `custom_error`

A generic helper in pre and post scripts that allows you to set **custom http status codes**. For instance, when you want to send an immediate response to the client from within a Lua script without further querying the backend, or after evaluating the response of the backend.

It stops the script and the pipe execution.

Example to throw a generic error (`500` status code ) with a message:

    custom_error("Something weird happened")

Or even changing the http status code (`418 I'm a teapot`)

    custom_error("I refuse to make any coffee, I'm a teapot!", 418)

## Sequence of execution

Request sequence:

1. Router  "source" files
2. Router  "pre" logic
3. Proxy   "source" files
4. Proxy   "pre" logic
5. Backend "source" files
6. Backend "pre" logic


The returned response then goes through:

7. Backend "post" logic
8. Proxy   "post" logic

## Examples

For the endpoint section:

    "extra_config": {
          "github.com/devopsfaith/krakend-lua/proxy": {
            "pre": "print('Lua proxy!'); local r = request.load(); r:headers('X-from-lua', '1234');"
          }
    }


For the backend section:

    "extra_config": {
          "github.com/devopsfaith/krakend-lua/proxy/backend": {
            "pre": "print('Showing body from backend-pre logic'); local r = request.load(); print(r:body(''));"
          }
    }

Setting a cookie:

    "extra_config": {
        "github.com/devopsfaith/krakend-lua/proxy": {
            "post": "local r = response.load(); r:headers('Set-Cookie', 'kkkey1='.. r:data('response'));",
            "allow_open_libs": true
        }
    }
