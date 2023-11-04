---
lastmod: 2021-12-07
date: 2021-12-07
linktitle: Docker artifact
title: Deploying KrakenD API Gateway with Docker
description: Learn how to deploy KrakenD API Gateway using Docker, enabling containerized deployments for efficient scaling and management
notoc: true
menu:
  community_current:
    parent: "110 Deployment and Go-Live"
weight: 20
---
If you use containers, the recommended approach is to write your own `Dockerfile` and deploy an **immutable artifact** (embedding the config).

In its simplified form would be:
```Dockerfile
FROM {{< product image >}}:{{< product minor_version >}}
COPY krakend.json /etc/krakend/krakend.json
# Uncomment with Enterprise image:
# COPY LICENSE /etc/krakend/LICENSE
```

{{< note title="Volume or copy?" type="question" >}}
Even though you can use the official container directly and attach the configuration mounting an external volume (or ConfigMap in Kubernetes), a custom image with your configuration copied inside has benefits. It guarantees that you can do safe rollbacks and have effective testing and debugging. If you break something at any point, you only need to deploy the previous container, while if you use a volume, you are exposed to downtime or impossible scaling until you fix it.
{{< /note >}}

A more real-life example illustrates below a combination of the `check` command with a multi-stage build to compile a [flexible configuration](/docs/configuration/flexible-config/) in a `Dockerfile`:

```docker
FROM {{< product image >}}:{{< product minor_version >}} as builder
ARG ENV=prod

COPY krakend.tmpl .
COPY config .

## Save temporary file to /tmp to avoid permission errors
RUN FC_ENABLE=1 \
    FC_OUT=/tmp/krakend.json \
    FC_PARTIALS="/etc/krakend/partials" \
    FC_SETTINGS="/etc/krakend/settings/$ENV" \
    FC_TEMPLATES="/etc/krakend/templates" \
    krakend check -d -t -c krakend.tmpl --lint

FROM {{< product image >}}:{{< product minor_version >}}
COPY --from=builder --chown=krakend:nogroup /tmp/krakend.json .
# Uncomment with Enterprise image:
# COPY LICENSE /etc/krakend/LICENSE
```

The `Dockerfile` above has two stages:

1. The copy of all templates and intermediate files to end with a `check` command that compiles the template `krakend.tmpl` and any included sub-template inside. The command outputs (thanks to `FC_OUT`) the result into a `/tmp/krakend.json` file.
2. The `krakend.json` file from the previous build is the only addition to the final Docker image.

The example `Dockerfile` above assumes that you have a file structure like this:

    .
    ├── config
    │   ├── partials
    │   ├── settings
    │   │   ├── prod
    │   │   │   └── env.json
    │   │   └── test
    │   │       └── env.json
    │   └── templates
    │       └── some.tmpl
    ├── Dockerfile
    └── krakend.tmpl

If you want to try this code, you can either download a [working Flexible Config example](https://github.com/krakend/examples/tree/main/3.flexible-configuration), or generate an **empty skeleton** like this:
```bash
mkdir -p config/{partials,settings,templates}
mkdir -p config/settings/{prod,test}
touch config/settings/{prod,test}/env.json
touch Dockerfile
touch krakend.tmpl
```

Now the only missing step to generate the image, is to build it, making sure that the environment argument matches our folder inside the `settings` folder:

{{< terminal title="Docker build" >}}
docker build --build-arg ENV=prod -t mykrakend .
{{< /terminal >}}

The resulting image, including your configuration, weighs around `80MB`.