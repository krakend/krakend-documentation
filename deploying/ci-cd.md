---
lastmod: 2021-12-7
date: 2021-12-7
linktitle: CI/CD integration
title: Automated KrakenD deployments with CI/CD
menu:
  community_current:
    parent: "110 Deploying"
weight: 20
---
KrakenD operates with its single binary and your associated configuration. Therefore, your build process or CI/CD pipeline only needs to ensure that the configuration file is correct. These are a few recommendations to a safes KrakenD deployment:

1. Make sure the configuration file is valid
2. Optional - On Docker, generate an immutable docker image
3. Optional - Run integration tests
4. Deploy the new configuration

There are several ways to automate KrakenD deployments, but **you must always test your configuration** before applying it in production. You'll find a few notes that might help you automate this process in this document.

## Making sure the configuration is valid
The `check` command is a must in any **CI/CD pipeline** or pre-deploy process to ensure you don't put a broken setup in production that results in downtime. The `check` command lets you find broken configurations before going live. Add a line like the following in your release process:

{{< terminal title="Recommended file check for CI/CD" >}}
krakend check -t -d -c /path/to/krakend.json
{{< /terminal >}}

The previous command will stop the pipeline (exit 1) if it fails or continue if the configuration is correct.

[Read more about the `check` command](/docs/commands/check/)

## Generating a Docker artifact
If you use containers, the recommended approach is to write your own `Dockerfile` and deploy an **immutable artifact** (embedding the config).

In its simplified form would be:
{{< highlight Dockerfile >}}
FROM devopsfaith/krakend
COPY krakend.json /etc/krakend/krakend.json
{{< /highlight >}}

{{< note title="Volume or copy?" type="question" >}}
Even though you can use the official container directly and attach the configuration mounting an external volume (or ConfigMap in Kubernetes), a custom image with your configuration copied inside has benefits. It guarantees that you can do safe rollbacks and have effective testing and debugging. If you break something at any point, you only need to deploy the previous container, while if you use a volume, you are exposed to downtime or impossible scaling until you fix it.
{{< /note >}}

A more real-life example illustrates below a combination of the `check` command with a multi-stage build to compile a [flexible configuration](/docs/configuration/flexible-config/) in a `Dockerfile`:

{{< highlight docker >}}
FROM devopsfaith/krakend as builder
ARG ENV=prod

COPY krakend.tmpl .
COPY config .

## Save temporary file to /tmp to avoid permission errors
RUN FC_ENABLE=1 \
    FC_OUT=/tmp/krakend.json \
    FC_PARTIALS="/etc/krakend/partials" \
    FC_SETTINGS="/etc/krakend/settings/$ENV" \
    FC_TEMPLATES="/etc/krakend/templates" \
    krakend check -d -t -c krakend.tmpl

FROM devopsfaith/krakend
COPY --from=builder --chown=krakend /tmp/krakend.json .
{{< /highlight >}}

The `Dockerfile` above has two stages:
 The `check` command compiles the template `krakend.tmpl` and any included sub-template inside and outputs (`FC_OUT`) the resulting `/tmp/krakend.json` file.
The `krakend.json` file is the only addition to the final Docker image.

The example above assumes you have a file structure like this:

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

Generate the skeleton above with:
{{< highlight bash >}}
mkdir -p config/{partials,settings,templates}
mkdir -p config/settings/{prod,test}
touch Dockerfile krakend.tmpl
{{< /highlight >}}

Now the only missing step to generate the image, is to build it, making sure that the environment argument matches our folder inside the `settings` folder:

{{< terminal title="Docker build" >}}
docker build --build-arg ENV=prod -t mykrakend .
{{< /terminal >}}

The resulting image including your configuration will weight around `80MB`.

## Running the integration tests
TO-DO

## Deploy the new configuration
Use a blue-green deployment or similar strategy to make sure there is no downtime when you roll-out new KrakenD versions.

Read here if you use [Kubernetes](/docs/deploying/kubernetes/)