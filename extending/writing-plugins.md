---
lastmod: 2019-01-15
date: 2019-01-14
toc: true
linktitle: Writing custom plugins
title: Writing custom plugins
weight: 1
skip_header_image: true
menu:
  community_current:
    parent: "150 Custom Plugins and Middleware"
images:
- /images/documentation/krakend-plugins.png
---
All different types of plugins let you freely implement your logic without restrictions. To start using your own plugins make sure to write them implementing the right interface and compile them respecting the requirements.

{{< note title="Introduction to plugins" type="info" >}}
Before getting your hands dirty, read the [introduction to plugins](/docs/extending/introduction/) for understanding the different types of plugins you can use.
{{< /note >}}


## Plugin requirements
Writing, compiling and using plugins need to comply with the following list:

- **Right interface**: Your plugin implements the proper interface (see each plugin type)
- **Same go version**: You compile the plugin using the same Go version KrakenD was compiled with
- **Same architecture/platform**: You compile the plugin using the same architecture where KrakenD will run. E.g., you cannot compile a plugin in a Mac and use it in a Docker container).
- **Same import versions**: When using external libraries if KrakenD also uses them, they must be in the same version.
- **Register and inject** your plugins in the configuration.

## Compiling plugins
As your custom plugins need to match the Go and libraries versions used to build KrakenD, you have to make sure your plugin is compatible by checking your `go.sum` file with the command `check-plugin` (read the documentation)

{{< terminal title="Term" >}}
krakend check-plugin -v 1.17.0 -s ../myplugin/go.sum
1 incompatibility(ies) found...
go
	have: 1.17.0
	want: 1.16.4
{{< /terminal >}}

Once your plugin is written with the plugin type interface you have chosen, compile it in the same architecture type as follows:

{{< terminal title="Go compilation">}}
go build -buildmode=plugin -o yourplugin.so
{{< /terminal >}}

### Compiling plugins inside Docker
The official KrakenD container uses **[Alpine](https://hub.docker.com/_/alpine)** as the base image. Therefore, when compiling your plugins inside a Docker that extends the official image with a go-alpine builder, you will need to install at least the following requirements in the `Dockerfile`:

{{< highlight Dockerfile >}}
RUN apk add make gcc musl-dev
{{< /highlight >}}

## Registering and injecting plugins
Your plugin is coded and ready to use, and now you want to use it. There are two phases:

- Registering the plugin
- Injecting the plugin in a specific place

### Loading the plugin
KrakenD registers plugins **during startup** according to its plugin configuration:

{{< highlight json >}}
{
    "version": 3,
    "plugin": {
        "pattern":".so",
        "folder": "/opt/krakend/plugins/"
    }
}
{{< /highlight >}}
Add the `plugin` keyword at the root of your configuration to let KrakenD know the rules to register plugins. The **mandatory** options you need to declare are:

- `folder` (*string*): The directory path in the filesystem where all the plugins you **want to load** are. **MUST END IN SLASH**. The folder can be a relative or absolute path, but end it in slash!. E.g: KrakenD Enterprise stores the plugins in the path  `/opt/krakend/plugins/`.
- `pattern` (*string*): The pattern narrows down the folder's contents and acts as a filter. It represents the **substring that must be present** in the plugin name to load. KrakenD will load any plugin with a `.so` extension in the example above. You could also use any prefix or suffix to match the content or even the full name of a single plugin. For instance, if you want to load the rewrite plugin, use `"pattern":"krakend-rewrite.so"`, or use `-prod.so` to load all safe production plugins ending with that suffix. The rules are up to you.

Place the plugin in the folder you have declared in the configuration and start KrakenD. At this point and with the previous configuration, you have **registered plugins during startup**, and you should see a line early in the logs when starting KrakenD. The log lines depend on the type of plugin you have chosen, an example:

    INFO [SERVICE: Handler Plugin] Total plugins loaded: 1

### Checking the plugin registration
When the service starts, **KrakenD doesn't know the type of plugin** you have coded until it tries to register it, and **it will try to register it as all known types**. When using a `DEBUG` log level you will see this activity in the logs.

In most cases, you will create your plugin for a single type, but it doesn't mean that you cannot implement more than one type of plugin per file. The registration attempts are reflected in the logs, and you will see log lines that could look like errors, but they are not!

{{< note title="Log line 'symbol X not found in plugin Y'" type="error" >}}
This logline is not an error (it's a DEBUG message). It tells you that your plugin cannot register itself as one of the other two types of plugins you are not implementing. It's all good.
{{< /note >}}

For example, let's see how loading three different plugins are logged into KrakenD:

- `client-example.so`  (An [HTTP client plugin](/docs/extending/http-client-plugins/))
- `server-example.so` (An [HTTP server plugin](/docs/extending/http-server-plugins/))
- `request-modifier.so` (A [request/response modifier plugin](/docs/extending/plugin-modifiers/))

In the logs, we will see how each plugin fails to register as the rest of the types they don't implement:

{{< highlight txt "hl_lines=4-5 8-9 12-13">}}
Parsing configuration file: krakend.json
▶ INFO Listening on port: 8080
▶ DEBUG [PLUGIN: client-example] Logger loaded
▶ DEBUG [SERVICE: Executor Plugin] plugin #1 (request-modifier.so): plugin: symbol ClientRegisterer not found in plugin mytest
▶ DEBUG [SERVICE: Executor Plugin] plugin #2 (server-example.so): plugin: symbol ClientRegisterer not found in plugin myserver
▶ INFO [SERVICE: Executor Plugin] Total plugins loaded: 1
▶ DEBUG [PLUGIN: server-example] Logger loaded
▶ DEBUG [SERVICE: Handler Plugin] plugin #0 (client-example.so): plugin: symbol HandlerRegisterer not found in plugin myclient
▶ DEBUG [SERVICE: Handler Plugin] plugin #1 (request-modifier.so): plugin: symbol HandlerRegisterer not found in plugin mytest
▶ INFO [SERVICE: Handler Plugin] Total plugins loaded: 1
▶ DEBUG [PLUGIN: request-modifier] Logger loaded
▶ DEBUG [SERVICE: Modifier Plugin] plugin #0 (client-example.so): plugin: symbol ModifierRegisterer not found in plugin myclient
▶ DEBUG [SERVICE: Modifier Plugin] plugin #2 (server-example.so): plugin: symbol ModifierRegisterer not found in plugin myserver
▶ INFO [SERVICE: Modifier Plugin] Total plugins loaded: 1
{{< /highlight >}}

The `INFO` log level tells you what is going on, but notice how the highlighted `DEBUG` messages fail to register for the type they are not. This is expected.

### Using the plugin
At this point, KrakenD has registered the plugin and is ready to use. The next step is to inject the plugin somewhere in the configuration. The configuration entry depends entirely on the type of plugin you are using and what you have coded.

Below there is a sample configuration for an HTTP server fake plugin:

{{< highlight json >}}
{
    "version": 3,
    "plugin": {
        "pattern":".so",
        "folder": "/opt/krakend/plugins/"
    },
    "extra_config": {
        "plugin/http-server": {
            "name": ["my_plugin", "another_plugin_maybe" ],
            "my_plugin": {
                "some_flag_in_my_plugin": true
            }
        }
    }
}
{{< /highlight >}}