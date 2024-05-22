---
lastmod: 2022-06-14
date: 2021-05-21
toc: true
linktitle: HTTP client plugins
title: HTTP Client Plugins for KrakenD API Gateway
description: Learn how to extend the functionality of KrakenD API Gateway by utilizing HTTP client plugins, enabling customized communication with external services
weight: 110
menu:
  community_current:
    parent: "180 Extending with custom code"
meta:
  since: 2.0
  namespace:
  - plugin/http-client
images:
- /images/documentation/diagrams/plugin-type-client.mmd.svg
- /images/documentation/krakend-plugins.png
- /images/documentation/http-client-plugin.png
---
The **HTTP client** plugins execute in the proxy layer when KrakenD tries to reach your backends for content. They allow you to intercept, transform, and manipulate the requests **before they hit your backend services**, and their way back. It is the perfect time to modify the request before it reaches the backend.

You cannot chain HTTP client plugins, limiting them to one plugin per backend connection, and replace the default KrakenD's HTTP client.

{{< note title="HTTP client components will stop working" type="info" >}}
An HTTP client is a terminator. It means that it is the last executor in the KrakenD pipe. When you add an HTTP client plugin, **you replace KrakenD's default client** with your own. It means some built-in functionality in the default HTTP client won't exist unless you code it.

More specifically, if you inject your plugin, you don't have [Client Credentials](/docs/authorization/client-credentials/) or [Backend Cache](/docs/backends/caching/), and you won't have telemetry at the HTTP backend level.
{{< /note >}}

![http handler plugin](/images/documentation/http-client-plugin.png)

## HTTP client interface
{{< note title="Writing plugins" type="tip" >}}
Read the introduction to [writing plugins](/docs/extending/writing-plugins/) for compilation and configuration options if you haven't done it yet.
{{< /note >}}

To start with a *hello world* for your first plugin, you have to implement the plugin client interface by copying the example provided in the [Go documentation](https://godoc.org/github.com/luraproject/lura/transport/http/client/plugin)

### Example: Building your first client plugin
The easiest way to demonstrate how HTTP client plugins work is with a Hello World plugin. So let's start by creating a new Go project named `krakend-client-example`:

    mkdir krakend-client-example
    cd krakend-client-example
    go mod init krakend-client-example

Now we have to create a file `main.go` with the content below:

```go
// SPDX-License-Identifier: Apache-2.0

package main

import (
	"context"
	"encoding/json"
	"errors"
	"fmt"
	"html"
	"io"
	"net/http"
)

// ClientRegisterer is the symbol the plugin loader will try to load. It must implement the RegisterClient interface
var ClientRegisterer = registerer("krakend-client-example")

type registerer string

var logger Logger = nil

func (registerer) RegisterLogger(v interface{}) {
	l, ok := v.(Logger)
	if !ok {
		return
	}
	logger = l
	logger.Debug(fmt.Sprintf("[PLUGIN: %s] Logger loaded", ClientRegisterer))
}

func (r registerer) RegisterClients(f func(
	name string,
	handler func(context.Context, map[string]interface{}) (http.Handler, error),
)) {
	f(string(r), r.registerClients)
}

func (r registerer) registerClients(_ context.Context, extra map[string]interface{}) (http.Handler, error) {
	// check the passed configuration and initialize the plugin
	name, ok := extra["name"].(string)
	if !ok {
		return nil, errors.New("wrong config")
	}
	if name != string(r) {
		return nil, fmt.Errorf("unknown register %s", name)
	}

	// check the cfg. If the modifier requires some configuration,
	// it should be under the name of the plugin. E.g.:
	/*
	   "extra_config":{
	       "plugin/http-client":{
	           "name":"krakend-client-example",
	           "krakend-client-example":{
	               "path": "/some-path"
	           }
	       }
	   }
	*/

	// The config variable contains all the keys you hace defined in the configuration:
	config, _ := extra["krakend-client-example"].(map[string]interface{})

	// The plugin will look for this path:
	path, _ := config["path"].(string)
	logger.Debug(fmt.Sprintf("The plugin is now hijacking the path %s", path))

	// return the actual handler wrapping or your custom logic so it can be used as a replacement for the default http handler
	return http.HandlerFunc(func(w http.ResponseWriter, req *http.Request) {

		// The path matches, it has to be hijacked and no call to the backend happens.
		// The path is the the call to the backend, not the original request by the user.
		if req.URL.Path == path {
			w.Header().Add("Content-Type", "application/json")
			// Return a custom JSON object:
			res := map[string]string{"message": html.EscapeString(req.URL.Path)}
			b, _ := json.Marshal(res)
			w.Write(b)
			logger.Debug("request:", html.EscapeString(req.URL.Path))

			return
		}

		// If the requested path is not what we defined, continue.
		resp, err := http.DefaultClient.Do(req)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}

		// Copy headers, status codes, and body from the backend to the response writer
		for k, hs := range resp.Header {
			for _, h := range hs {
				w.Header().Add(k, h)
			}
		}
		w.WriteHeader(resp.StatusCode)
		if resp.Body == nil {
			return
		}
		io.Copy(w, resp.Body)
		resp.Body.Close()

	}), nil
}

func main() {}

type Logger interface {
	Debug(v ...interface{})
	Info(v ...interface{})
	Warning(v ...interface{})
	Error(v ...interface{})
	Critical(v ...interface{})
	Fatal(v ...interface{})
}
```

The plugin above aborts the request and replies by printing a `Hello, %q` without passing the request to the backend. It is a simple example, but it shows the necessary structure to start working with plugins.

If you look now closely at the plugin code, notice that the loader looks for the symbol `ClientRegisterer` and, if found, the loader checks if the symbol implements the `plugin.Registerer` interface. Once the plugin is validated, the loader registers handlers from the plugin by calling the exposed `RegisterClients` method.

With the `main.go` file saved, it's time to build and test the plugin. If you added more code and external dependencies, you must run a `go mod tidy` before the compilation, which is unnecessary for this example.

For compiling Go plugins, the flag `-buildmode=plugin` is required. The command is:

{{< terminal >}}
go build -buildmode=plugin -o krakend-client-example.so .
{{< /terminal >}}

If you are using Docker and want to load your plugin on Docker, compile it in the [Plugin Builder](/docs/extending/writing-plugins/#plugin-builder) for easier integration.

{{< terminal title="Build your plugin" >}}
docker run -it -v "$PWD:/app" -w /app \
{{< product image_plugin_builder >}}:{{< product latest_version >}} \
go build -buildmode=plugin -o krakend-client-example.so .
{{< /terminal >}}

There is no output for this command. Now you have a file `krakend-client-example.so`, the KrakenD binary has to side load. Remember that you cannot use this binary in a different architecture (e.g., compiling the binary in Mac and loading it in a Docker container).

The plugin is ready to use! You can now load your plugin in the configuration. Add the `plugin` and `extra_config` entries in your configuration. Here's an example of `krakend.json`:

```json
{
  "version": 3,
  "plugin": {
    "pattern": ".so",
    "folder": "./krakend-client-example/"
  },
  "endpoints": [
    {
      "endpoint": "/test/{id}",
      "backend": [
        {
          "host": [
            "http://localhost:8080"
          ],
          "url_pattern": "/__debug/{id}",
          "extra_config": {
            "plugin/http-client": {
              "name": "krakend-client-example",
              "krakend-client-example": {
                "path": "/__debug/hijack-me"
              }
            }
          }
        }
      ]
    }
  ]
}
```

Start the server with `krakend run -dc krakend.json`. When you run the server, the expected output (with `DEBUG` log level) is:

    Parsing configuration file: krakend.json
    yyyy/mm/dd hh:mm:ss KRAKEND ERROR: [SERVICE: Logging] Unable to create the logger: getting the extra config for the krakend-gologging module
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [SERVICE: Plugin Loader] Starting loading process
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [PLUGIN: krakend-client-example] Logger loaded
    yyyy/mm/dd hh:mm:ss KRAKEND INFO: [SERVICE: Executor Plugin] Total plugins loaded: 1
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [SERVICE: Handler Plugin] plugin #0 (krakend-client-example/krakend-client-example.so): plugin: symbol HandlerRegisterer not found in plugin krakend-client-example
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [SERVICE: Modifier Plugin] plugin #0 (krakend-client-example/krakend-client-example.so): plugin: symbol ModifierRegisterer not found in plugin krakend-client-example
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [SERVICE: Plugin Loader] Loading process completed
    yyyy/mm/dd hh:mm:ss KRAKEND INFO: Starting the KrakenD instance
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [ENDPOINT: /test/:id] Building the proxy pipe
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [BACKEND: /__health] Building the backend pipe
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: The plugin is now hijacking the path /hijack-me
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [BACKEND: /__health] Injecting plugin krakend-client-example
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [ENDPOINT: /test/:id] Building the http handler
    yyyy/mm/dd hh:mm:ss KRAKEND DEBUG: [ENDPOINT: /test/:id][JWTSigner] Signer disabled
    yyyy/mm/dd hh:mm:ss KRAKEND INFO: [ENDPOINT: /test/:id][JWTValidator] Validator disabled for this endpoint
    yyyy/mm/dd hh:mm:ss KRAKEND INFO: [SERVICE: Gin] Listening on port: 8080
    ...

Let's take a closer look at the log. First, notice that the plugin tried registering itself for each plugin type (`[SERVICE: Executor Plugin]`, `[SERVICE: Handler Plugin]`, and `[SERVICE: Modifier Plugin]`), but we are only building an Executor Plugin in this case.

As we are implementing only one of the types, the other two types will fail to load (`symbol not found`). The logline is expected and is not an error but just an informational `DEBUG` message.

The essential lines are:
- `[PLUGIN: krakend-client-example] Logger loaded` printed by the plugin logger we introduced in our code telling us that the plugin is loaded
- The `[SERVICE: Executor Plugin] Total plugins loaded: 1` telling us there is one type of plugin for this type
- `[BACKEND: /__health] Injecting plugin krakend-client-example` telling us that the plugin is loaded AND injected by the configuration.

If you see these lines, you did great! Your plugin is working.

To test the plugin, request the test endpoint `/test/{id}`. If you request a path not declared in the configuration, like `/test/normal,` the plugin will execute the request. If you request to `/test/hijack-me,`, then the plugin will respond with the content `Hello, /hijack-me.`

{{< terminal title="Delegate the request" >}}
curl http://localhost:8080/test/normal
{"message":"pong"}
{{< /terminal >}}


{{< terminal title="Hijack the request" >}}
curl http://localhost:8080/test/hijack-me
{"message":"/__debug/hijack-me"}
{{< /terminal >}}

The plugin is now working.
