---
lastmod: 2019-01-15
old_version: true
date: 2019-01-14
toc: true
linktitle: Injecting plugins
title: Loading and injecting plugins
weight: 2
skip_header_image: true
menu:
  community_v2.0:
    parent: "150 Custom Plugins and Middleware"
images:
- /images/documentation/krakend-plugins.png
---
Your plugin is already developed and ready to use, and now you want to use it. There are two phases:

- Loading the plugin
- Injecting the plugin in a specific place

## Loading the plugin
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

- `folder` (*string*): The directory path in the filesystem where all the plugins you **want to load** are. The folder can be a relative or absolute path!. E.g: KrakenD Enterprise stores the plugins in the path  `/opt/krakend/plugins/`.
- `pattern` (*string*): The pattern narrows down the folder's contents and acts as a filter. It represents the **substring that must be present** in the plugin name to load. KrakenD will load any plugin with a `.so` extension in the example above. You could also use any prefix or suffix to match the content or even the full name of a single plugin. For instance, if you want to load the rewrite plugin, use `"pattern":"krakend-rewrite.so"`, or use `-prod.so` to load all safe production plugins ending with that suffix. The rules are up to you.

Place the plugin in the folder you have declared in the configuration and start KrakenD. At this point and with the previous configuration, you have **registered plugins during startup**, and you should see a line early in the logs when starting KrakenD. The log lines depend on the type of plugin you have chosen, an example:

    INFO [SERVICE: Handler Plugin] Total plugins loaded: 1

### Checking the plugin registration
When the service starts, **KrakenD doesn't know the type of plugin** you have coded until it tries to register it, and **it will try to register it as all known types**. When using a `DEBUG` log level you will see this activity in the logs.

In most cases, you will create your plugin for a single type, but it doesn't mean that you cannot implement more than one type of plugin per file. The registration attempts are reflected in the logs, and you will see log lines that could look like errors, but they are not!

{{< note title="Log line 'symbol X not found in plugin Y'" type="error" >}}
This logline is not an error (it's a DEBUG message). It tells you that your plugin cannot register itself as one of the other types of plugins you are not implementing. **It's all good**.
{{< /note >}}

For example, let's see how loading three different plugins are logged into KrakenD:

- `client-example.so`  (An [HTTP client plugin](/docs/v2.0/extending/http-client-plugins/))
- `server-example.so` (An [HTTP server plugin](/docs/v2.0/extending/http-server-plugins/))
- `request-modifier.so` (A [request/response modifier plugin](/docs/v2.0/extending/plugin-modifiers/))

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

## Injecting the plugin
At this point, KrakenD has registered the plugin and is ready to use. The next step is to inject the plugin somewhere in the configuration. The configuration entry depends entirely on the type of plugin you are using and what you have coded.

### Injecting HTTP server plugins
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

- `name` (*list*): The list of all the plugins you want to inject.
- After the name, you can include any key that your plugin requires to load configuration. In the example `my_plugin`.

### Injecting HTTP client plugins
HTTP client plugins live in the backend's `extra_config`, and you can declare one plugin per backend.

{{< highlight json >}}
{
    "version": 3,
    "plugin": {
        "pattern":".so",
        "folder": "/opt/krakend/plugins/"
    },
    "endpoints":[
        {
            "endpoint": "/foo",
            "backend": [
                {
                    "url_pattern": "/__debug",
                    "extra_config": {
                        "plugin/http-client": {
                            "name": "your-plugin"
                        }
                    }
                }
            ]
        }
    ]
}
{{< /highlight >}}

- `name` (*string*): The name of the plugin you want to inject.

### Injecting request and response modifier plugins
You can place the request/modifier plugins at the `endpoint` level or the `backend` level. In both cases you can inject several plugins that are used in the order that you declare them.

{{< highlight json >}}
{
  "version": 3,
  "plugin": {
    "pattern":".so",
    "folder": "/path/to/your/plugin/folder/"
  },
  "endpoints": [
    {
      "endpoint": "/github/orgs/{org}",
      "extra_config":{
        "plugin/req-resp-modifier":{
          "name":["your-plugin"]
        }
      },
      "backend":[
        {
          "url_pattern": "/orgs/{org}",
          "extra_config":{
            "plugin/req-resp-modifier":{
              "name":["your-plugin"]
            }
          }
        }
      ]
    }
  ]
}