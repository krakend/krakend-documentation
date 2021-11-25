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

- **Right interface**: Your plugin implements the right interface (see each plugin type)
- **Same go version**: You compile the plugin using the same Go version KrakenD was compiled with
- **Same architecture/platform**: You compile the plugin using the same architecture where KrakenD will run. E.g.: you cannot compile a plugin in a Mac and use it in a Docker container).
- **Same import versions**: When using external libraries, if they are also used by KrakenD they have to be in the same version
- **Register and inject** your plugins in the configuration.

## Compiling plugins
To respect Go and libraries versions see what versions where used to compile the KrakenD version you have chosen to use. For that, use the **dependencies finder**. If you already have a resulting `go.sum` file, validate it with the **go.sum validator**:

{{< button-group >}}
{{< button url="https://plugin-tools.krakend.io/" text="Dependencies finder" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
</svg>{{< /button >}}
{{< button url="https://plugin-tools.krakend.io/validate" type="inversed" >}}go.sum validator{{< /button >}}
{{< /button-group >}}

Once your plugin is written with the plugin type interface you have chosen, compile it in the same architecture type as follows:

{{< terminal title="Go compilation">}}
go build -buildmode=plugin -o yourplugin.so
{{< /terminal >}}

### Compiling plugins inside Docker
The official KrakenD container uses **[Alpine](https://hub.docker.com/_/alpine)** as the base image. When compiling your plugins inside a Docker that extends the official image with a go-alpine builder you will need to install at least the following requirements in the `Dockerfile`:

{{< highlight Dockerfile >}}
RUN apk add make gcc musl-dev
{{< /highlight >}}

## Registering and injecting plugins
Your plugin is coded and ready to use and now you want to use it. There are two phases:

- Registering the plugin
- Injecting the plugin in a specific place

### Loading the plugin
KrakenD registers plugins **during startup** according to its plugin configuration:

{{< highlight json >}}
{
    "version": 2,
    "plugin": {
        "pattern":".so",
        "folder": "/opt/krakend/plugins/"
    }
}
{{< /highlight >}}
Add the `plugin` keyword at the root of your configuration to let KrakenD know the rules to register plugins. The **mandatory** options you need to declare are:

- `folder` (*string*): The directory path in the filesystem where all the plugins you **want to load** are. **MUST END IN SLASH**. The folder can be a relative or absolute path, but end it in slash!. E.g: KrakenD Enterprise stores the plugins in the path  `/opt/krakend/plugins/`.
- `pattern` (*string*): The pattern narrows down the contents of the folder and acts as a filter. It represents the **substring that must be present** in the plugin name to load. In the example above, any plugin with a `.so` extension will be loaded. You could also use any prefix or suffix to match the content or even the full name of a single plugin. For instance, if you just want to load the rewrite plugin, use `"pattern":"krakend-rewrite.so"`, or use `-prod.so` to load all production safe plugins ending with that sufix. The rules are up to you.

At this point and with the previous configuration, you have **registered plugins during startup**, and you should see a line early in the logs when starting KrakenD. 

### Using the plugin
At this point KrakenD has registered the plugin and can be used. The next step is to inject the plugin somewhere in the configuration. The configuration entry depends entirely on the type of plugin you are using, and what you have coded.

This a sample configuration for an HTTP server fake plugin:

{{< highlight json >}}
{
    "version": 2,
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