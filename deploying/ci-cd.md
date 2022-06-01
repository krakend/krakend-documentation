---
lastmod: 2021-12-07
date: 2021-12-07
linktitle: CI/CD integration
title: Automated KrakenD deployments with CI/CD
notoc: true
menu:
  community_current:
    parent: "110 Deployment and Go-Live"
weight: 20
---
KrakenD operates with its single binary and your associated configuration. Therefore, your build process or CI/CD pipeline only needs to ensure that the configuration file is correct. These are a few recommendations to a safes KrakenD deployment:

1. Make sure the configuration file is valid. When using Flexible Configuration, generate the final `krakend.json` using `FC_OUT` as the final artifact
2. Optional - [Generate an immutable docker image](/docs/deploying/docker/)
3. Optional - [Run integration tests](/docs/developer/integration-tests/)
4. Deploy the new configuration

There are several ways to automate KrakenD deployments, but **you must always test your configuration** before applying it in production. You'll find a few notes that might help you automate this process in this document.

For the first step, the `check` command is a must in any **CI/CD pipeline** or pre-deploy process to ensure you don't put a broken setup in production that results in downtime. The `check` command lets you find broken configurations before going live. Add a line like the following in your release process:

{{< terminal title="Recommended file check for CI/CD" >}}
krakend check -t -d -c /path/to/krakend.json
{{< /terminal >}}

The command above will stop the pipeline (`exit 1`) if it fails or continue if the configuration is correct. Make sure to always place it in your build/deploy process.

[Read more about the `check` command](/docs/configuration/check/)