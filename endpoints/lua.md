---
lastmod: 2019-09-15
date: 2019-09-15
linktitle:  Lua scripting
title: Transformations using Lua scripting
weight: 90
source: https://github.com/devopsfaith/krakend-lua
since: 1.0
menu:
  documentation:
    parent: endpoints
---
Scripting with Lua is an additional choice to extend your business logic, and is compatible with the rest of options such as [CEL](/docs/endpoints/common-expression-language-cel/), [Martian](/docs/endpoints/martian/), or other Go plugins and middlewares.

If you are more familiar with Lua than Go, this module can help you solve exceptional cases that need solution using a little bit of scripting. The introduction of Lua scripts in your Gateway does not require to recompile KrakenD, but unlike Go, Lua scripts are compiled and evaluated in real-time.

For performance-first users, a Go plugin delivers better results than a Lua script.

# Supported Lua types (cheatsheet)
When running Lua scripts on KrakenD, there are two different types you can use in their coding. Depending on the place of the pipe you want to place the script you can use a `proxy` type or a `router` type:

    End User <--[router]--> KrakenD <--[proxy]--> Services

These two types are described as follows:

- Router: The router layer is what happens between the end-user and KrakenD
- Proxy: The proxy layer is between KrakenD and your services


## `proxy` type
Use this type when you need to intercept requests and responses between KrakenD and your services.

### Request
Scripts that need to modify a request that KrakenD is about to do against the backend services.


- `load` (*Static*)
- `method` (*Dynamic*)
- `path` (*Dynamic*)
- `query` (*Dynamic*)
- `url` (*Dynamic*)
- `params` (*Dynamic*)
- `headers` (*Dynamic*)
- `body` (*Dynamic*)

Example: Access the request getter with `req:url()` and the setter with `req:url("foo")`.

### Response
Scripts that need to modify a request that KrakenD is about to get from the backend services.

- `load` (*Static*)
- `isComplete` (*Dynamic*)
- `statusCode` (*Dynamic*)
- `data` (*Dynamic*)
- `headers` (*Dynamic*)
- `body` (*Dynamic*)

## `router` type
Use this type when you need to script the router layer, traffic between end-users and KrakenD.


### ctx

- `load` (*Static*)
- `method` (*Dynamic*)
- `query` (*Dynamic*)
- `url` (*Dynamic*)
- `params` (*Dynamic*)
- `headers` (*Dynamic*)
- `body` (*Dynamic*)

# Additional helpers (cheatsheet)
The following helpers are available in your scripts:

## `table`

- `get` (*Dynamic*)
- `set` (*Dynamic*)
- `len` (*Dynamic*)

## `list`

- `get` (*Dynamic*)
- `set` (*Dynamic*)
- `len` (*Dynamic*)

## `http_response`

- `new` (*Static*)
- `statusCode` (*Dynamic*)
- `headers` (*Dynamic*)
- `body` (*Dynamic*)
