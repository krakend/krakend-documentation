---
lastmod: 2022-10-21
old_version: true
date: 2019-09-15
linktitle:  Lua scripts
title: Lua Scripting
description: Explore the power of Lua scripting in KrakenD API Gateway, allowing you to customize and extend the behavior of your API gateway
weight: 200
menu:
  community_v2.9:
    parent: "180 Extending with custom code"
meta:
  since: 1.0
  source: https://github.com/krakend/krakend-lua
  namespace:
  - "modifier/lua-proxy"
  - "modifier/lua-endpoint"
  - "modifier/lua-backend"
  scope:
  - service
  - endpoint
  - backend
  - async_agent
  log_prefix:
  - "[ENDPOINT: /foo][Lua]"
  - "[BACKEND: /foo][Lua]"
---

Scripting with Lua allows you to extend your business logic and make **transformations on requests and responses**. The Lua module is compatible with the rest of components such as [CEL](/docs/v2.9/endpoints/common-expression-language-cel/), [Martian](/docs/v2.9/backends/martian/), or other [Go plugins](/docs/v2.9/extending/) and middlewares.

The introduction of Lua scripts in your Gateway does not require recompiling KrakenD, but unlike Go, Lua scripts are interpreted in real-time. If you are new to Lua, see [Lua Documentation](https://www.lua.org/).

{{< note title="Lua vs Go Plugins" type="note" >}}
A [Go plugin](/docs/v2.9/extending/) delivers much more speed and power than a Lua script for performance-first seeking users, but requires a little bit more work as you need to compile your plugins and side-load them on KrakenD.
{{< /note >}}

## Configuration

You can add your Lua scripts under the `extra_config` at the service level, the `endpoint` level or the `backend` level. You can choose **three different namespaces** (explained below):

- `"modifier/lua-endpoint"` (endpoint and service level)
- `"modifier/lua-proxy"` (endpoint level)
- `"modifier/lua-backend"` (backend level)

The configuration options are:

```json
{
    "extra_config": {
        "modifier/lua-proxy": {
            "sources": [
                "file1.lua",
                "./relative/path/file2.lua",
                "/etc/krakend/absolute/path.lua"
            ],
            "md5": {
                "file1.lua": "49ae50f58e35f4821ad4550e1a4d1de0"
            },
            "pre": "print('Hi from pre!'); my_file1_function()",
            "post": "print('Hi from post!'); my_file1_function()",
            "live": false,
            "allow_open_libs": false,
            "skip_next": false
        }
    }
}
```

{{< schema version="v2.9" data="modifier/lua.json" >}}

## Configuration placement and sequence of execution
When running Lua scripts, you can place them at the `proxy` level, or the `router` level:

![lua namespaces](/images/documentation/diagrams/lua.mmd.svg)

These two places have the following considerations:

- **Router** (at `endpoint`'s `extra_config` or service level): Communication between the end-user and KrakenD. You can inspect and modify the **request** of the user.
  - With `"modifier/lua-endpoint"`you can modify the **HTTP request context** early in the transport layer. However, KrakenD has not converted the request into an internal request just yet.
  - With `"modifier/lua-proxy"`you can modify the internal KrakenD request before reaching all backends in the endpoint and modify the response **AFTER the merge** of all backends.
- **Proxy** (at `backend`'s `extra_config`): Communication between KrakenD and your services. For both the **request** and the **response**.
  - With `"modifier/lua-backend"`you can modify the internal KrakenD request before reaching a particular backend and change its response **BEFORE is passed for the merge** of backends at the endpoint level.

In a request/response execution, this is how the different namespaces for Lua placement work:

![Lua - Sequence of execution](/images/documentation/diagrams/lua-2.seq.mmd.svg)

## Functions for Proxy

You can use the following Lua functions to access and manipulate requests and responses in `"modifier/lua-proxy"` and `"modifier/lua-backend"` namespaces.

### Request functions (`request`)
If you have a script that needs access to the request, use the `request` object in Lua. The request is set when KrakenD is about to do a call to the backend services.

{{< note title="Using client headers and querystrings" >}}
When **client headers** or **query strings** are needed in a script, remember to add them under [`input_headers`](/docs/v2.9/endpoints/parameter-forwarding/#headers-forwarding) or [`input_query_strings`](/docs/v2.9/endpoints/parameter-forwarding/#query-string-forwarding) accordingly.
{{< /note >}}

The `request` functions are:

*   `load()` (_Static_): The constructor to view and manipulate requests. E.g.: `local r = request.load()`. **Notice that the rest of the functions rely on this one**.
*   `method()` (_Dynamic_): Getter that retrieves the method of the request. E.g.: `r:method()` could return a string `GET`.
*   `method(value)` (_Dynamic_): Setter that changes the method of the request. E.g.: `r:method('POST')`.
*   `path()` (_Dynamic_): Getter that retrieves the path of the request. E.g.: `r:path()` could return a string `/foo/var`.
*   `path(value)` (_Dynamic_): Setter that changes the path of the request. E.g.: `r:path('/foo/var')`. It does not have any effect when you use `modifier/lua-backend`.
*   `query()` (_Dynamic_): Getter that retrieves the query string of the request, URL encoded. E.g.: `r:query()` could return a string `?foo=var&vaz=42`.
*   `query(value)` (_Dynamic_): Setter that changes the query of the request. E.g.: `r:query('?foo=var&vaz=42')`. It does not have any effect when you use `modifier/lua-backend`, but you can still set the `url()` without query strings.
*   `url()` (_Dynamic_): Getter that retrieves the full URL string of the request, including the host and path. E.g.: `r:url()` could return a string `http://domain.com/api/test`. The URL might be empty depending on the step where this information is requested, as the URL is a calculated field just before performing the request to the backend.
*   `url(value)` (_Dynamic_): Setter that changes the URL of the request. E.g.: `r:url('http://domain.com/api/test')`. Changing the value before the `url` is calculated will result in KrakenD overwriting its value. Although available, it does not have any effect when you use it `modifier/lua-proxy`.
*   `params(param)` (_Dynamic_): Getter that retrieves the `{params}` of the request as defined in the endpoint. E.g.: For an endpoint `/users/{user}` the function `r:params('User')` could return a string `alice`. **The parameters must have the first letter capitalized**.
*   `params(param,value)` (_Dynamic_): Setter that changes the params of the request. It does not have any effect when you use `modifier/lua-backend`. E.g.: `r:params('User','bob')`. **The parameters must have the first letter capitalized**.
*   `headers(header)` (_Dynamic_): Getter that retrieves the headers of the request as allowed to pass (by `input_headers`) in the endpoint. E.g.: `r:headers('Accept')` could return a string `*/*`.
*   `headers(header,value)` (_Dynamic_): Setter that changes the headers of the request. E.g.: `r:headers('Accept','*/*')`. If the value is `nil`, the header is removed from the request.
*   `headerList(header)` (_Dynamic_): Getter that returns a `luaList` with the multiple values of a request header. E.g.: `req:headerList("X-Added")`.
*   `headerList(header,list)` (_Dynamic_): Setter that changes the headers with multiple values of the request. E.g.: `local n = luaList.new();n:set(0, "first"); n:set(1, "second");req:headerList("X-Added", n)`.
*   `body()` (_Dynamic_): Getter that retrieves the body of the request sent by the user. E.g.: `r:body()` could return a string `{"foo": "bar"}`.
*   `body(value)` (_Dynamic_): Setter that changes the body of the request. E.g.: `r:body('{"foo": "bar"}')`.

### Response functions (`response`)

Scripts that need to modify a request that KrakenD that just got from the backend service.

*   `load()` (_Static_): The constructor to view and manipulate responses. E.g.: `local r = response.load()`. **Notice that the rest of the functions rely on this one**.
*   `isComplete()` (_Dynamic_): Getter that returns a boolean if the response from the backend (or a merge of backends) succeeded with a `20x` code, and completed successfully before the timeout. E.g.: `r:isComplete()` returns `true` or `false`.
*   `isComplete(bool)` (_Dynamic_): Setter that allows you to mark a response as completed. It will change the internal `X-KrakenD-Complete: true` header. E.g.: `r:isComplete(true)` tells KrakenD everything went OK (even not true).
*   `statusCode()` (_Dynamic_): Getter that retrieves the response status code when you use `no-op` encoding. You will always get a `0` in the other cases. E.g.: `r:statusCode()` returns an integer `200`.
*   `statusCode(integer)` (_Dynamic_): Setter that allows you to set a new status for the response. E.g.: `r:statusCode(301)`. It only works in `no-op` endpoints.
*   `data()` (_Dynamic_): Getter that returns a Lua table with all the parsed data from the response. It only works if you don't use `no-op` encoding.
*   `data(table)` (_Dynamic_): While **there is no setter for the `data` function** you can set individual items of the data. For instance, do a `local responseData = r:data()` first, and then set individual items with `responseData:set("key", value)` instead. It only works if you don't use `no-op` encoding.
*   `headers(header)` (_Dynamic_): Getter that retrieves one header from the response when you use `no-op` encoding. In the rest of the responses, you will always get an empty string `''`. E.g.: `r:headers('Content-Type')` returns an integer `application/json`.
*   `headers(header,value)` (_Dynamic_): Setter that allows you to replace or set a new header for the response when you use `no-op` encoding. E.g.: `r:headers('Content-Type', 'application/json')`. If the value is `nil`, the header is removed from the response.
*   `headerList(header)` (_Dynamic_): Getter that returns a `luaList` with the multiple values of a response header. E.g.: `r:headerList("X-Added")`.
*   `headerList(header,list)` (_Dynamic_): Setter that changes the headers with multiple values of the response. E.g.: `local n = luaList.new();n:set(0, "first"); n:set(1, "second");resp:headerList("X-Added", n)`.
*   `body()` (_Dynamic_): Getter that retrieves the body of the response when you use encoding `no-op`. E.g.: `r:body()` could return a string `{"foo": "bar"}`.
*   `body(value)` (_Dynamic_): Setter that changes the body of the response when you use encoding `no-op`. E.g.: `r:body('{"foo": "bar"}')`.


## Functions for Router

Use this type when you need to script the router layer, traffic between end-users, and KrakenD with the `"modifier/lua-endpoint"` namespace.

### Context functions (`ctx`)

*   `load()` (_Static_): The constructor to view and manipulate requests. E.g.: `local c = ctx.load()`. **Notice that the rest of the functions rely on this one**.
*   `method()` (_Dynamic_): Getter that retrieves the method of the request. E.g.: `c:method()` could return a string `GET`.
*   `method(value)` (_Dynamic_): Setter that changes the method of the request. E.g.: `c:method('POST')`.
*   `query(key)` (_Dynamic_): Getter that retrieves the query string of the request, URL encoded. E.g.: `c:query('foo')` could return a string `var` for `?foo=var&vaz=42`.
*   `query(key,value)` (_Dynamic_): Setter that changes the query of the request. E.g.: `c:query('foo','var')`.
*   `url()` (_Dynamic_): Getter that retrieves the URL string of the request (path and parameters). E.g.: `c:url()` could return a string `/api/test?foo=bar`. The URL might be empty depending on the step where this information is requested, as the URL is a calculated field just before performing the request to the backend.
*   `url(value)` (_Dynamic_): Setter that changes the url of the request. E.g.: `c:url('/api/test?foo=bar')`. Changing the value before the `url` is calculated will result in KrakenD overwriting its value.
*   `params(param)` (_Dynamic_): Getter that retrieves the `{params}` of the request as defined in the endpoint. E.g.: For an endpoint `/users/{user}` the function `c:params('User')` could return a string `alice`. **The parameters must have the first letter capitalized**.
*   `params(param,value)` (_Dynamic_): Setter that changes the params of the request. E.g.: `c:params('User','bob')`. **The parameters must have the first letter capitalized**.
*   `headers(header)` (_Dynamic_): Getter that retrieves the headers of the request as allowed to pass (by `input_headers`) in the endpoint. E.g.: `c:headers('Accept')` could return a string `*/*`.
*   `headers(header,value)` (_Dynamic_): Setter that changes the headers of the request. E.g.: `c:headers('Accept','*/*')`.
*   `headerList(header)` (_Dynamic_): Getter that returns a `luaList` with the multiple values of a context header. E.g.: `c:headerList("X-Added")`.
*   `headerList(header,list)` (_Dynamic_): Setter that changes the headers with multiple values of the request. E.g.: `local n = luaList.new();n:set(0, "first"); n:set(1, "second");ctx:headerList("X-Added", n)`.
*   `body()` (_Dynamic_): Getter that retrieves the body of the request sent by the user. E.g.: `c:body()` could return a string `{"foo": "bar"}`.
*   `body(value)` (_Dynamic_): Setter that changes the body of the request. E.g.: `c:body('{"foo": "bar"}')`.
*   `host()` (_Dynamic_): Getter that retrieves the `Host` header of the request sent by the user. E.g.: `c:host()` could return a string `api.domain.com`.
*   `host(value)` (_Dynamic_): Setter that changes the host header of the request. E.g.: `c:host('api.domain.com')`.


## Lua helpers
Now you know where to put the Lua code according to what you want to do, and how to access and modify the requests and responses. In addition, the following helper functions are brought by KrakenD to extend the possibilities of your scripts without using third parties:

### Nil helper
The `nil` in Lua is destructive in tables, because it is used to **remove elements**. For instance, if you have a table with an index `foo`, and you set it to `nil`, the element from the table is destroyed. When you want to preserve this `foo` index but emptying it's value you must use the Go `nil` value instead.

*   `luaNil.new()`: Returns a native Go `nil` value. This is useful if you want to write `nil` values in a `luaTable` or a `luaList`.

### Tables helper (`table`)
To work with associative arrays on Lua you have the following functions:

*   `new()` (_Static_): Returns a new table. E.g.: `local t = luaTable.new()`
*   `keys()` (_Dynamic_): Returns the sorted key names of a table. E.g.: `local k = t:keys()`
*   `keyExists(key)`(_Dynamic_): Returns a boolean. True when `key` is a member of the table or false otherwise. E.g.: `local r = response.load(); local responseData = r:data(); responseData:keyExists('some_key')`
*   `get(key)` (_Dynamic_): Retrieves the value of a key inside the table. E.g.: `local r = response.load(); local responseData = r:data(); responseData:get('key')`
*   `set(key,value)` (_Dynamic_): Adds or replaces a key in the table. E.g.: `local r = response.load(); local responseData = r:data(); responseData:set('key',value)`
*   `len()` (_Dynamic_): Returns the length of the whole table so you can iterate over it. E.g.: `local r = response.load(); local responseData = r:data(); local length = responseData:len()`
*   `del(key)` (_Dynamic_): Deletes a key from a table. E.g.: `local r = response.load(); local responseData = r:data(); responseData:del('key')`

An example of Lua script that gets a field `source_result` from a table and sets a new key `result` accordingly by reading the response text (decorator pattern):

```lua
function post_proxy_decorator( resp )
  local responseData = resp:data()
  local responseContent = responseData:get("source_result")
  local message = responseContent:get("message")

  local c = string.match(message, "Successfully")

  if not not c
  then
    responseData:set("result", "success")
  else
    responseData:set("result", "failed")
  end
end
```


Another example that iterates all fields in the response and filters out everything but a specific field would be:

```lua
-- This function filters out all keys in the response data except the 'keyToPreserve'.
function filter(req, resp)
    -- Get the data from the response object
    local responseData = resp:data()

    -- Define the key that should not be removed
    local keyToPreserve = 'KeepThisField'

    -- Get all keys from the response data
    local allKeys = responseData:keys()

    -- Loop through all the keys
    for i = 0, allKeys:len() - 1 do
        local currentKey = allKeys:get(i)
        print("Found key:", currentKey)

        -- If the current key is not the one to preserve, remove it
        if currentKey ~= keyToPreserve then
            responseData:del(currentKey)
            print("Removing key:", currentKey)
        end
    end
end
```
The `filter` function would be called as follows in the configuration:

```json
{
    "modifier/lua-backend": {
      "allow_open_libs": true,
      "sources": ["./filter.lua"],
      "post": "filter(request.load(), response.load())"
    }
}
```

### Collections helper (`list`)

*   `new()` (_Static_): Returns a new list. E.g.: `local t = luaList.new()`
*   `get(key)` (_Dynamic_): Retrieves the value of a key inside the list. E.g.: `local r = response.load(); local responseData = r:data(); local l = responseData:get('collection'); l:get(1)` gets the item at position 1 from the list.
*   `set(key,value)` (_Dynamic_): Adds or replaces a key in the list. E.g.: `local r = response.load(); local responseData = r:data(); local l = responseData:get('collection'); l:set(1,value)` sets the value of position `1`.
*   `len()` (_Dynamic_): Returns the length of the whole list so you can iterate over it. E.g.: `local r = response.load(); local responseData = r:data(); local l = responseData:get('collection'); l:len()`
*   `del(key)` (_Dynamic_): Deletes an offset from a list. E.g.: `local r = response.load(); local responseData = r:data(); local l = responseData:get('collection'); l:del(1)`

Example of Lua code that iterates the items under the array `collection` and also uses sets and deletes tables:

```lua
-- A function that receives a response object through response.load()
function post_proxy( resp )
  local data = {}
  local responseData = resp:data()
  local col = responseData:get("collection")
  local size = col:len()

  -- Sets a new attribute "total" in the response with the number of elements in the array
  responseData:set("total", size)

  local paths = {}
  for i=0,size-1 do
    local element = col:get(i)
    local t = element:get("path")
    table.insert(paths, t)
  end
  responseData:set("paths", paths)
  responseData:del("collection")
end
```



### Advanced helpers
The [Lua advanced helpers](/docs/enterprise/endpoints/lua-advanced-helpers/) ({{< badge >}}Enterprise{{< /badge >}}) provide a powerful way to extend the capabilities of your Lua scripts, enabling advanced functionality with optimal performance. These helpers, which run natively in Go, significantly enhance Lua's efficiency by offloading heavy-lifting operations to Go and passing the results back to Lua. This architecture ensures both speed and reliability while empowering developers with advanced tools.

The helpers cover a wide range of functionalities, from **debugging** to encoding and decoding JSON, YAML, XML, and CSV formats. Advanced features such as base64 encoding/decoding, hashing (e.g., SHA, MD5, FNV), and time manipulation expand the scope of Lua scripting, making it easier to handle complex scenarios. By leveraging these native helpers, you can process data, transform backend responses, and implement custom logic with minimal overhead and maximum performance.


{{< button-group >}}
{{< button url="/docs/enterprise/endpoints/lua-advanced-helpers/" type="inversed" >}}Enterprise Lua Helpers{{< /button >}}
{{< /button-group >}}


### Making additional requests (`http_response`)
The `http_response` helper allows you to make an additional HTTP request and access its response. Is is available on:

- `"modifier/lua-proxy"` (endpoint level)
- `"modifier/lua-backend"` (backend level)
- `"modifier/lua-endpoint"` (endpoint level, **Enterprise only** until CE 2.10)

Notice that you **cannot** use it in `modifier/lua-endpoint`.

*   `new(url)` (_Static_): Constructor. Sets the URL you want to call and makes the request. E.g.: `local r = http_response.new('http://api.domain.com/test')`. **Notice that the rest of the functions rely on this one**. The constructor accepts 1, 3, or 4 arguments, respectively. See examples below.
*   `statusCode()` (_Dynamic_): Getter for the status code of the response. E.g.: `r:statusCode()` could return `200`
*   `headers(header)` (_Dynamic_) : Getter for a specific header of the response. E.g.: `r:headers('Content-Type')` could return `application/json`
*   `headerList(header)` (_Dynamic_): Getter that returns a `luaList` with the multiple values of a header. E.g.: `r:headerList("X-Added")`.
*   `body()` (_Dynamic_): Getter for the full response body.
*   `close()` (_Dynamic_): Closes the HTTP connection to free resources. Although it will be done automatically later by KrakenD, a better approach is to close the resource as soon as you don't need it anymore.

```lua
local url = 'http://api.domain.com/test'

-- Constructor with 1 parameter
local r = http_response.new(url)
print(r:statusCode())
print(r:headers('Content-Type'))
print(r:body())
r:close()

-- Constructor with 3 parameters
local r = http_response.new(url, "POST", '{"foo":"bar"}')
print(r:statusCode())
print(r:headers('Content-Type'))
print(r:body())
r:close()

-- Constructor with 4 parameters
local r = http_response.new(url, "POST", '{"foo":"bar"}', {["foo"] = "bar", ["123"] = "456"})
print(r:statusCode())
print(r:headers('Content-Type'))
print(r:body())
r:close()
```

### Set custom HTTP status codes (`custom_error`)

A generic helper in pre and post-scripts that allows you to set **custom HTTP status codes**. For instance, when you want to send an immediate response to the client from within a Lua script without further querying the backend, or after evaluating the response of the backend.

It stops the script and the pipe execution. Its signature is as follows:

```lua
custom_error(message, status_code, content_type)
```
The parameters of the function are:
- `message` (*string*): The message you want to send
- `status_code` (*integer*): Optional. The HTTP status code
- `content_type` (*string*): Optional. The `Content-Type` to return

Example of throwing a generic error (`500` status code ) with a message:

```lua
custom_error("Something weird happened")
```

Or even changing the HTTP status code (`418 I'm a teapot`)

```lua
custom_error("I refuse to make any coffee, I'm a teapot!", 418)
```


## Debugging scripts
Because Lua scripts evaluate during the request execution, you won't see any problems until a request triggers the scripts you have configured. While developing, the errors will be decorated so you can find easily which file and line number are the responsible ones for an error. For instance, have a look at the following log:

```log
attempt to index a non-table object(function) with key 'really_bad' (bad-code.lua:5)
```

The ending `(bad-code.lua:5)` tells you that this error was initiated at the script `bad-code.lua` on **line number** `5`. To understand the errors, check the Lua language as these come from the Lua engine directly.

## Language limitations
Because Lua runs sandboxed in a **virtual machine**, there is certain stuff you cannot do. When you include files, you should limit your logic to creating functions and referencing them in different places.

In addition to that, the usage of modules would require KrakenD to push them in advance to the virtual machine, so user-created modules are not possible. **Package managers like LuaRocks are not supported** either, because they inject modules globally in the system, not in the virtual machine.

{{< note title="Modules not supported" type="error" >}}
You **cannot create your modules** neither use `require` to import them.
{{< /note >}}

Finally, if you need to use the standard library, you will need to add in the configuration the flag `allow_open_libs`.

## Lua examples in different pipes
The following snippets show how to add Lua code in different sections.

### Lua in the service for all endpoints
An example setting a common header in the request to all endpoints.

```json
  {
    "version": 3,
    "extra_config": {
        "modifier/lua-endpoint": {
          "pre": "print('Lua service!'); local c = ctx.load(); c:headers('X-from-lua', '1234');"
        }
    }
  }
```

### Lua in the endpoint
An example of setting lua scripts in three different stages that modify headers.

```json
  {
    "endpoint": "/set-a-header",
    "extra_config": {
        "modifier/lua-endpoint": {
          "pre": "print('Modifies the HTTP request'); local c = ctx.load(); c:headers('X-from-lua-endpoint', '1234');"
        },
        "modifier/lua-proxy": {
          "@comment-pre": "Modifies the internal proxy request before the split to all backends",
          "pre": "print('Lua proxy pre modifier'); local r = request.load(); r:headers('X-from-lua-proxy', '1234');",
          "@comment-post": "Modifies the response after merging all backends",
          "post": "print('Lua proxy post modifier'); local r = response.load(); r:headers('X-from-lua-proxy', '1234');",
        }
    }
  }
```
- The `modifier/lua-endpoint` hits first as it modifies the HTTP request directly. Can't modify the response (`post`).
- The `pre` in `modifier/lua-proxy` modifies the internal request before it's sent to each of the backends you have configured in the endpoint. Backends see an extra header.
- The `post` in `modifier/lua-proxy` modifies the response after having merged all the contents from all backends.

### Lua in the backend
An example showing how to **print the backend response** in the console.
```json
{
    "extra_config": {
          "modifier/lua-backend": {
            "pre": "print('Backend response, pre-logic:'); local r = request.load(); print(r:body());"
          }
    }
}
```

Another example **setting a cookie from Lua**:
```json
{
    "extra_config": {
        "modifier/lua-backend": {
            "post": "local r = response.load(); r:headers('Set-Cookie', 'key1='.. r:data('response'));",
            "allow_open_libs": true
        }
    }
}
```
