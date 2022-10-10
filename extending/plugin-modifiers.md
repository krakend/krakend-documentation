---
lastmod: 2021-05-21
date: 2021-05-21
toc: true
linktitle: Req/resp modifier plugins
title: Request and response modifier plugins
weight: 30
menu:
  community_current:
    parent: "150 Custom Plugins and Middleware"
meta:
  since: 2.0
  namespace:
  - plugin/req-resp-modifier
images:
- /images/documentation/krakend-plugins.png
- /images/documentation/request-response-plugin.png

---
The **request and response modifier plugins** are a type of KrakenD customization that allow you to code your own business logic directly on requests and responses in a simple and extensible way. These plugins complement the **handler plugins**, and the **client executor plugins** and avoid their limitations and **extra overload**.

The injecting of the modifiers is placed at the beginning of the proxy pipe (just after the router layer) and before the request executor (where the executor plugins are injected). The first one can modify the incoming request and the final response, and it's configured at the endpoint level; the second one can modify the request and responses concerning the backend that declares it. Since all the modifiers are executed at the proxy pipe, no extra encoding/decoding cycles are required.


The **request and response modifier plugins are not middlewares, but modifier functions** that you can call **sequentially from a new middleware**. Request modifiers can only inspect and modify requests (and other cool things) and response modifiers, only responses.


KrakenD **executes the request and response modifiers in the order they are declared at the configuration**. See the [example](#example) below.

![http handler plugin](/images/documentation/request-response-plugin.png)

## Possibilities of req/resp modifiers
The following table shows what you can do with modifiers:

| Request modifiers | Response modifiers |
|-|-|
| 1. Request validation and complex checks | 1. Response validation and complex checks |
| 2. Request manipulation | 2. Response manipulation |
| 3. Request filtering | 3. Response filtering |
| 4. Request debugging | 4. Response debugging |
| 5. Complex workflows | 5. Complex workflows |
| 6. Coordinated rate-limiting and quota management | 6. Coordinated quota control (response size, service consumption, etc.)|

{{< note title="The encoding marks what you can manipulate in responses" type="warning" >}}
If your endpoint uses an `output_encoding` different than `no-op` you can work with `ResponseWrapper.Data()` and `ResponseWrapper.IsComplete()`. If you use `no-op` you can work with `ResponseWrapper.Io()`, `ResponseWrapper.StatusCode()`, and `ResponseWrapper.Headers()`.

If you set data in a different place than specified above, **it will be ignored**.
{{< /note >}}

## Example

{{< note title="Compatibility note" type="warning" >}}Go plugins are supported on Linux, FreeBSD, and macOS {{< /note >}}

The easiest way to demonstrate how the modifier plugins work is with the debugger plugin, so let's start with a new Go project by creating a new module with `go mod init your_package_name`, and adding a single `main.go` file with the minimal boilerplate.

{{< highlight go >}}
package main

import (
    "errors"
    "fmt"
    "io"
    "net/url"
    "github.com/luraproject/lura/v2/proxy"
)

func main() {}

func init() {
    fmt.Println(string(ModifierRegisterer), "loaded!!!")
}

// ModifierRegisterer is the symbol the plugin loader will be looking for. It must
// implement the plugin.Registerer interface
// https://github.com/luraproject/lura/blob/master/proxy/plugin/modifier.go#L71
var ModifierRegisterer = registerer("krakend-debugger")

type registerer string

// RegisterModifiers is the function the plugin loader will call to register the
// modifier(s) contained in the plugin using the function passed as argument.
// f will register the factoryFunc under the name and mark it as a request
// and/or response modifier.
func (r registerer) RegisterModifiers(f func(
    name string,
    factoryFunc func(map[string]interface{}) func(interface{}) (interface{}, error),
    appliesToRequest bool,
    appliesToResponse bool,
)) {
    f(string(r)+"-request", r.requestDump, true, false)
    f(string(r)+"-response", r.responseDump, false, true)
    fmt.Println(string(r), "registered!!!")
}
{{< /highlight >}}

As with other KrakenD plugins, the loader looks for a given symbol (in this case, "ModifierRegisterer") and, if found, the loader checks if the symbol implements the `plugin.Registerer` interface. Once the plugin is validated, the loader registers the modifiers from the plugin by calling the exposed `RegisterModifiers` method.

For the debugger plugin we'll register two different modifiers: `requestDump` and `responseDump`. The modifiers are registered under the same namespace so both will be injected with a single config line. In order to avoid weird dependency collisions, the `modifierFactory` signature uses basic types and `interface{}`, so some type assertion against the interfaces declared at the [`proxy` package](https://github.com/luraproject/lura/blob/master/proxy/plugin.go#L187-L209) are required. Include the interfaces into your plugin by adding the following lines:

{{< highlight go >}}
// RequestWrapper is an interface for passing proxy request between the krakend pipe
// and the loaded plugins
type RequestWrapper interface {
    Params() map[string]string
    Headers() map[string][]string
    Body() io.ReadCloser
    Method() string
    URL() *url.URL
    Query() url.Values
    Path() string
}

// ResponseWrapper is an interface for passing proxy response between the krakend pipe
// and the loaded plugins
type ResponseWrapper interface {
    Data() map[string]interface{}
    Io() io.Reader
    IsComplete() bool
    StatusCode() int
    Headers() map[string][]string
}
{{< /highlight >}}

After this minimal code repetition, implementing the modifier factories is an easy task. The factory must accept a configuration as a map and return the final modifier once it's ready. Depending on your requirements, the factory could access its dedicated configuration and do whatever logic is required by your scenario. This configuration section should use the plugin name as namespace (check the comments below).

{{< highlight go >}}
var unkownTypeErr = errors.New("unknow request type")

func (r registerer) requestDump(
    cfg map[string]interface{},
) func(interface{}) (interface{}, error) {
    // check the cfg. If the modifier requires some configuration,
    // it should be under the name of the plugin.
    // ex: if this modifier required some A and B config params
    /*
        "extra_config":{
            "plugin/req-resp-modifier":{
                "name":["krakend-debugger"],
                "krakend-debugger":{
                    "A":"foo",
                    "B":42
                }
            }
        }
    */

    // return the modifier
    fmt.Println("request dumper injected!!!")
    return func(input interface{}) (interface{}, error) {
        req, ok := input.(RequestWrapper)
        if !ok {
            return nil, unkownTypeErr
        }

        fmt.Println("params:", req.Params())
        fmt.Println("headers:", req.Headers())
        fmt.Println("method:", req.Method())
        fmt.Println("url:", req.URL())
        fmt.Println("query:", req.Query())
        fmt.Println("path:", req.Path())

        return input, nil
    }
}

func (r registerer) responseDump(
    cfg map[string]interface{},
) func(interface{}) (interface{}, error) {
    // check the cfg. If the modifier requires some configuration,
    // it should be under the name of the plugin.
    // ex: if this modifier required some A and B config params
    /*
        "extra_config":{
            "plugin/req-resp-modifier":{
                "name":["krakend-debugger"],
                "krakend-debugger":{
                    "A":"foo",
                    "B":42
                }
            }
        }
    */

    // return the modifier
    fmt.Println("response dumper injected!!!")
    return func(input interface{}) (interface{}, error) {
        resp, ok := input.(ResponseWrapper)
        if !ok {
            return nil, unkownTypeErr
        }

        fmt.Println("data:", resp.Data())
        fmt.Println("is complete:", resp.IsComplete())
        fmt.Println("headers:", resp.Headers())
        fmt.Println("status code:", resp.StatusCode())

        return input, nil
    }
}
{{< /highlight >}}

You can also refer [this example](https://github.com/luraproject/lura/blob/v2.0.1/proxy/plugin/tests/main.go) on how to update the request and [this example](https://github.com/luraproject/lura/blob/master/proxy/plugin.go#L130) on how to update the response.

### Building the plugin

With the `main.go` file complete, it's time to build and test the plugin. For compiling Go plugins, the flag `-buildmode=plugin` is required:

{{< terminal >}}
go build -buildmode=plugin -o krakend-debugger.so .
{{< /terminal >}}

If you are using Docker and wanting to load your plugin on Docker, compile it in the [Plugin Builder](/docs/extending/writing-plugins/#plugin-builder) for an easier integration.

{{< terminal title="Build your plugin" >}}
docker run -it -v "$PWD:/app" -w /app \
{{< product image_plugin_builder >}}:{{< product latest_version >}} \
go build -buildmode=plugin -o krakend-debugger.so .
{{< /terminal >}}

For the test, we'll build a small gateway with a single endpoint merging the responses from two different backends.

{{< highlight json >}}
{
  "version": 3,
  "port": 8080,
  "name": "KrakenD request and response modifier demo",
  "host": ["https://api.github.com"],
  "plugin": {
    "pattern":".so",
    "folder": "/path/to/your/plugin/folder/"
  },
  "endpoints": [
    {
      "endpoint": "/github/orgs/{org}",
      "backend":[
        {
          "url_pattern": "/orgs/{org}",
          "allow": [
            "avatar_url",
            "blog",
            "followers"
          ],
          "mapping": { "blog": "website" },
          "group": "org",
          "extra_config":{
            "plugin/req-resp-modifier":{
              "name":["krakend-debugger-request"]
            }
          }
        },
        {
          "url_pattern": "/orgs/{org}/repos",
          "mapping": { "collection": "repos" },
          "is_collection": true,
          "extra_config":{
            "plugin/req-resp-modifier":{
              "name":["krakend-debugger-response"]
            }
          }
        }
      ],
      "extra_config":{
        "plugin/req-resp-modifier":{
          "name": [
                "krakend-debugger-request",
                "krakend-debugger-response"
          ]
        }
      }
    }
  ]
}
{{< /highlight >}}

Notice the modifier names which needs to be combination of the modifier name and the string used while registering it in `RegisterModifiers`. Also these needs to be unique for request and response modifier.

If we send a request to the generated endpoint, we'll see the dumps for the three pairs of requests and responses at the console:

{{< terminal title="Test the code">}}
curl -i http://localhost:8080/github/orgs/krakendio
{{< /terminal >}}
