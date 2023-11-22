---
lastmod: 2023-04-12
date: 2023-04-12
linktitle: "Designer: Graphical Interface"
description: Discover the API Configuration Designer in KrakenD API Gateway, enabling intuitive and visual configuration management for your APIs
menu:
  community_current:
    parent: "000 Getting Started"
title: "API Configuration Designer in KrakenD API Gateway"
meta:
  source: https://github.com/krakend/krakendesigner
  since: 0.2
notoc: false
weight: 40
images:
- /images/builder_screenshot.png
skip_header_image: true
---
The [Designer](https://designer.krakend.io) is a UI that allows you to create KrakenD configurations from scratch or resume editing an existing one. It is a tool very useful in your **early contact with KrakenD**, as it helps you try functionalities without having to learn the different attributes of the configuration.

**The designer is a learning tool** more than an administration one. KrakenD configuration and administration is designed with **GitOps** in mind (under the version control system and released through CI/CD).

Combined with a [`:watch` container](/docs/developer/hot-reload/), you can **apply configuration changes automatically** in a development environment.

{{< button-group >}}
{{< button url="https://designer.krakend.io" text="Open designer" >}}<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
  <path stroke-linecap="round" stroke-linejoin="round" d="M15.042 21.672L13.684 16.6m0 0l-2.51 2.225.569-9.47 5.227 7.917-3.286-.672zM12 2.25V4.5m5.834.166l-1.591 1.591M20.25 10.5H18M7.757 14.743l-1.59 1.59M6 10.5H3.75m4.007-4.243l-1.59-1.59" />
</svg>
{{< /button >}}
{{< /button-group >}}


## Automatically applying changes to KrakenD
Suppose you have a **Chrome, Edge, or Opera** desktop browser. In that case, you can have the whole experience of editing in the browser and see the changes applied to your local development container without doing anything else.

{{< note title="The Designer uses experimental browser technology for file saving" type="info" >}}
While you can use the Designer in any major browser for editing files and downloading a copy, some experimental [browser capabilities](https://developer.mozilla.org/en-US/docs/Web/API/Window/showOpenFilePicker#browser_compatibility) allow you to open local files and **apply changes automatically on a [KrakenD Watch](/docs/developer/hot-reload/) server** by simply using the web.
{{< /note >}}

To use this, you need to:

- Start a container with the `:watch` tag
- Edit in the browser the file you have mounted in the volume.

From here, **every save will automatically apply the changes on KrakenD**.

You'll see a warning in the dashboard when your browser is not supported, or you are using a local copy without HTTPS.

The first time you attempt to save a file you have loaded from the disk, the browser will ask permission.

### Example of hot reload after browser change
Suppose you don't have an initial configuration. In that case, you can generate an initial one by clicking *Download* on the Designer without needing to configure anything yet, or you can paste this inside a new file, `krakend.json` instead:

```json
{
  "version": 3
}
```

Now that you have a fresh `krakend.json`, add a local `docker-compose.yaml` like this in the same folder if you are going to plug KrakenD into other containers locally:

```yml
version: "3"
services:
  krakend:
    image: {{< product image >}}:watch
    volumes:
      - ".:/etc/krakend"
    ports:
      - "8080:8080"
    command: [ "run", "-dc", "krakend.json" ]
```

Or do a `docker run` if you don't want a Docker compose:

```bash
docker run -it --rm -v "$PWD:/etc/krakend" {{< product image >}}:watch run -dc krakend.json
```

You can check that KrakenD is running by visiting its [health endpoint](/docs/service-settings/health/): http://localhost:8080/__health

Once KrakenD runs, the watcher follows changes happening in this folder. If you edit the file by hand, it will reload the new changes. But if you *Open* this file on the Designer, and save it, it will do it as well.

## Supported features
As KrakenD supports hundreds of features, it might be overwhelming to review all the documentation. Therefore, a tool that allows you to play in the browser is beneficial.

**The Designer supports *almost all* the functionality**, although advanced functionalities aren't in the interface. In any case, when this happens, even if you don't see them in the interface, they are kept in the final save if you loaded them.

The Designer does not support [flexible configuration](/docs/configuration/flexible-config/), as the browser cannot render Go templates of a complex directory structure.

The Designer supports editing Enterprise and Community features simultaneously. When you enable a single Enterprise feature, you will see a badge informing you about it.