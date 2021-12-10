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
KrakenD is a stateless application that needs a single configuration file attached to operate. Your build process or CI/CD pipeline should take the following steps to have a safer KrakenD deployment:

1. Make sure the configuration file is valid
2. Generate an immutable artifact (Docker image) - optional
3. Run integration tests - optional
4. Deploy the new configuration

Even there are several ways to automate KrakenD deployments, you should at least try your configuration before applying it in production. In this document you'll find a few notes that might help you automate this process.

## Making sure the configuration is valid
The `check` command is a must in any **CI/CD pipeline** or build process to make sure you don't deploy a broken configuration that results in downtime. The `check` command lets you find broken configurations before going live. Consider adding a line like the following in your release process:

{{< terminal title="Recommended file check for CI/CD" >}}
krakend check -t -d -c path/to/krakend.json
{{< /terminal >}}

[Read more about the `check` command](/docs/commands/check/)

## Generating a Docker artifact
When using `Dockerfile` artifacts, the recommended approach is to generate an **immutable artifact** over using volumes. Although you can use the official container and mount an external volume (or ConfigMap in Kubernetes) with the configuration, a custom image with your configuration copied inside allows you to go back to any previous state with safe rollbacks and debugging.

The following example illustrates a combination of the `check` command with a a multi-stage build to compile a [flexible configuration](/docs/configuration/flexible-config/) in a `Dockerfile`:

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

Notice how the lines above `check` that the configuration is valid using a starting template `krakend.tmpl` and output the compiled template into a `/tmp/krakend.json` file. This file is the only addition to the final Docker image.

It assumes that you have a file structure like this:

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

And that you build with something like:

{{< terminal title="Docker build" >}}
docker build --build-arg ENV=prod -t krakend-prod .
{{< /terminal >}}

## Running the integration tests
TO-DO

## Deploy the new configuration
Use a blue-green deployment or similar strategy to make sure there is no downtime when you roll-out new KrakenD versions.

Read here if you use [Kubernetes](/docs/deploying/kubernetes/)