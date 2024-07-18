---
lastmod: 2018-11-02
old_version: true
date: 2017-01-21
linktitle: Watch and hot reload
title: Hot Reload Feature
description: Explore the hot reload feature in KrakenD API Gateway, allowing dynamic configuration updates without service interruption for enhanced agility on development
weight: 80
#notoc: true
menu:
  community_v2.6:
    parent: "010 Configuration files"
---
When you want to **restart KrakenD automatically after saving** or changing files inside the configuration folder, the Docker image with the tag `:watch` can do this automatically for you.

It works whether you use [flexible configuration](/docs/v2.6/configuration/flexible-config/) or a single file (`krakend.json` or any other [supported format](/docs/v2.6/configuration/supported-formats/)).

The difference between the `:watch` image and `:latest` is that the KrakenD service wraps in a [reflex watcher](https://github.com/cespare/reflex). When it detects that the configuration has changed, it restarts the service again, applying the changes. This behavior is very convenient while developing as it allows you to test new changes without manually restarting KrakenD back and forth.

{{< note title="Use it in development" type="warning" >}}
The `:watch` Docker image is a development tool meant to speed up your initial KrakenD onboarding and testing, but it is **not designed as a production tool**. See below the risks of using `:watch` in production.
{{< /note >}}

## Watch usage
The `:watch` tag is a regular KrakenD Docker container and accepts the same options and flags as its `:latest` counterpart. Therefore, you should be able to replace `:watch` with the desired tag (e.g: `:2.6`) in production without further changes.

The container expects you to mount the configuration as a volume mapped to `/etc/krakend`. For instance:

{{< terminal title="Simple KrakenD watch" >}}
docker run --rm -v "$PWD:/etc/krakend" {{< product image >}}:watch
{{< /terminal >}}

When you use [flexible configuration](/docs/v2.6/configuration/flexible-config/), you can pass the environmental vars when starting the container. For instance:

{{< terminal title="Hot reload with Flexible Configuration on Community" >}}
docker run --rm -it -v "$PWD:/etc/krakend" \
    -e "FC_ENABLE=1" \
    -e "FC_OUT=compiled.json" \
    -e "FC_PARTIALS=/etc/krakend/partials" \
    -e "FC_SETTINGS=/etc/krakend/settings" \
    -e "FC_TEMPLATES=/etc/krakend/templates" \
    devopsfait/krakend:watch run -c krakend.tmpl
{{< /terminal >}}

{{< terminal title="Hot reload with Flexible Configuration on Enterprise" >}}
docker run --rm -it -v "$PWD:/etc/krakend" \
    -e "FC_CONFIG=/etc/krakend/flexible_config.json" \
    krakend/krakend-ee:watch run -c krakend.tmpl
{{< /terminal >}}

You can also integrate inside your IDE a watch to continuously check the configuration with the `check` command instead of `run`.

## Risks of hot-reloading in production

Because you might be tempted to use it in production, we don't recommend it because:

- It kills the server without a graceful option. If a user consumes an endpoint, the connection will be interrupted immediately, no matter if it ended or not.
- You might break the configuration after a save and leave the server unable to restart.
- KrakenD runs through a watch binary wrapper, hurting it, and will:
  - Decrease the performance (10 to 30% less throughput)
  - Decrease the stability of the binary (you might have undesired panics and unexpected exits)

### Why KrakenD needs restarting?
- **Performance**: This is the #1 reason, and strong enough. **KrakenD focuses on performance**, and during startup, calculates routes and decision trees, and the configuration is never read again.
- **GitOps**: We believe an API contract must go under the version control system and, after that, be released (and through CI/CD if possible), but never altered at runtime by throwing curls or whatever (Stick to [POLA](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)).
- **Consistency**: Imagine yourself distributing configuration updates in an ecosystem like the one KrakenD is offering: **independent, non-coordinated, multi-region, and stateless**. It would require you to develop complex strategies to ensure consistency that is far out of the scope of an API Gateway.
- **It happens in other servers, too**: You are used to this behavior from Varnish, Apache, Nginx, Mysql, or any other major service. All of them are reloaded when configuration changes and the Internet is still running. [Blue/green deployment](/docs/v2.6/deploying/#use-bluegreen-or-similar-deployment-strategy) and similar approaches were invented to ensure there is never downtime even if you change the service.
