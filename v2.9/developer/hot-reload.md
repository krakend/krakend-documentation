---
lastmod: 2025-04-09
old_version: true
date: 2017-01-21
linktitle: Watch and hot reload
title: Hot Reload Feature
description: Explore the hot reload feature in KrakenD API Gateway, allowing dynamic configuration updates without service interruption for enhanced agility on development
weight: 80
#notoc: true
menu:
  community_v2.9:
    parent: "010 Configuration files"
---
The Docker tag `:watch` is a **development-phase image** that allows you to **restart KrakenD automatically after saving** or changing files inside the configuration folder, although **is not recommended for production workloads**. It works whether you use [flexible configuration](/docs/v2.9/configuration/flexible-config/) or a single file (`krakend.json` or any other [supported format](/docs/v2.9/configuration/supported-formats/)).

The watch is a regular KrakenD binary wrapped in a [reflex watcher](https://github.com/cespare/reflex). When it detects that any piece inside the configuration has changed, it restarts the service, applying the changes. This behavior is very convenient while developing as it allows you to test new changes without manually restarting KrakenD back and forth

{{< note title="You must build the tag `:watch` on the Community Edition" type="info" >}}
Since KrakenD became an [official Docker image](https://hub.docker.com/_/krakend), the `:watch` tag is no longer available in the Docker registry **for the open source edition**. To use `:watch`, you **must build** the Docker tag as described below, but Enterprise users don't need to build this image and can continue working as always as `:watch` is ready to [`docker pull`](/docs/enterprise/developer/hot-reload/).
{{< /note >}}

## Building the watch image
To use the `:watch` image on the Open Source edition, [clone the `krakend-watch` repository](https://github.com/krakend/krakend-watch/) and do the `docker build` as explained in its README:

```bash
git clone https://github.com/krakend/krakend-watch.git
cd krakend-watch
docker build -t {{< product image >}}:watch
```
If you want to pin a specific KrakenD version, replace the `:latest` string on the `Dockerfile` with your desired version before building.

After the build completes, you will be able to use `{{< product image >}}:watch` normally.

## Watch usage
The container expects you to mount the configuration as a volume mapped to `/etc/krakend`. For instance:

{{< terminal title="Simple KrakenD watch" >}}
docker run --rm -v "$PWD:/etc/krakend" {{< product image >}}:watch
{{< /terminal >}}

When you use [flexible configuration](/docs/v2.9/configuration/flexible-config/), you can pass the environment variables when starting the container. For instance:

{{< terminal title="Hot reload with Flexible Configuration" >}}
docker run --rm -it -v "$PWD:/etc/krakend" \
    -e "FC_ENABLE=1" \
    -e "FC_OUT=compiled.json" \
    -e "FC_PARTIALS=/etc/krakend/partials" \
    -e "FC_SETTINGS=/etc/krakend/settings" \
    -e "FC_TEMPLATES=/etc/krakend/templates" \
    {{< product image >}}:watch run -c krakend.tmpl
{{< /terminal >}}

You can also integrate inside your IDE a watch to continuously check the configuration with the `check` command instead of `run`.

## Risks of hot-reloading in production

As we said, in the introduction, this tag is for development. Because you might be tempted to use it in production, we don't recommend it because:

- It kills the server without a graceful option. If a user consumes an endpoint, the connection will be interrupted immediately, no matter if it ended or not.
- You might break the configuration after a save and leave the server unable to restart.
- KrakenD runs through a watch binary wrapper, hurting it, and will:
  - Decrease the performance (10 to 30% less throughput)
  - Decrease the stability of the binary (you might have undesired panics and unexpected exits)

### Why KrakenD needs restarting?
- **Performance**: This is the #1 reason, and strong enough. **KrakenD focuses on performance**, and during startup, calculates routes and decision trees, and the configuration is never read again.
- **GitOps**: We believe an API contract must go under the version control system and, after that, be released (and through CI/CD if possible), but never altered at runtime by throwing curls or whatever (Stick to [POLA](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)).
- **Consistency**: Imagine yourself distributing configuration updates in an ecosystem like the one KrakenD is offering: **independent, non-coordinated, multi-region, and stateless**. It would require you to develop complex strategies to ensure consistency that is far out of the scope of an API Gateway.
- **It happens in other servers, too**: You are used to this behavior from Varnish, Apache, Nginx, Mysql, or any other major service. All of them are reloaded when configuration changes and the Internet is still running. [Blue/green deployment](/docs/v2.9/deploying/#use-bluegreen-or-similar-deployment-strategy) and similar approaches were invented to ensure there is never downtime even if you change the service.
