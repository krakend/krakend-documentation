---
lastmod: 2021-12-07
old_version: true
date: 2021-12-07
linktitle: CI/CD integration
title: "CI/CD Deployment on the API Gateway"
description: Streamline your API deployments with KrakenD using continuous integration and continuous deployment (CI/CD) practices. Follow our comprehensive guide to automate your deployment pipeline.
notoc: true
menu:
  community_v2.5:
    parent: "190 Deployment and Go-Live"
weight: 20
---
KrakenD operates with its single binary and your associated configuration. Therefore, your build process or CI/CD pipeline only needs to ensure that the configuration file is correct. These are a few recommendations to a safer KrakenD deployment:

1. Make sure the configuration file is valid. When using Flexible Configuration, generate the final `krakend.json` using `FC_OUT` as the final artifact
2. Optional - Ensure ther are no severe security problems using the [`audit` command](/docs/v2.5/configuration/audit/).
3. Optional - [Generate an immutable docker image](/docs/v2.5/deploying/docker/)
4. Optional - [Run integration tests](/docs/v2.5/developer/integration-tests/)
5. Deploy the new configuration

There are several ways to automate KrakenD deployments, but **you must always test your configuration** before applying it in production. You'll find a few notes that might help you automate this process in this document.

For the first step, the `check` command is a must in any **CI/CD pipeline** or pre-deploy process to ensure you don't put a broken setup in production that results in downtime. The `check` command lets you find broken configurations before going live. Add a line like the following in your release process:

{{< terminal title="Recommended file check for CI/CD" >}}
krakend check --lint -t -d -c /path/to/krakend.json
{{< /terminal >}}

The command above will stop the pipeline (`exit 1`) if it fails or continue if the configuration is correct. Make sure to always place it in your build/deploy process.

[Read more about the `check` command](/docs/v2.5/configuration/check/)

## Gitlab pipeline example
Here you have an example pipeline for Gitlab. You can add more steps like the audit command, but it serves as an example that you can start with. You can also adapt this workflow to other CI/CD systems by looking at the actions performed:
```yaml
# This file is a template, and needs editing before it works on your project.
# In your first run you should check in what
# Build a Docker image with CI/CD and push to the GitLab registry.
# Docker-in-Docker documentation: https://docs.gitlab.com/ee/ci/docker/using_docker_build.html
#
# This template uses one generic job with conditional builds
# for the default branch and all other (MR) branches.
stages:
  - license
  - test
  - build

# This step is only needed in Enterprise
check-license:
  stage: license
  image: ubuntu
  script:
    - apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y openssl bc
    - WARN_IN_DAYS=30
    - LICENSE_FILE_PATH="./LICENSE"
    - NOW=$(date +%s)
    - EXPIRATION=$(openssl x509 -in $LICENSE_FILE_PATH -text -noout -dates | grep notAfter | sed -e 's#notAfter=##')
    - EXPIRATION_TIMESTAMP=$(date -d "$EXPIRATION" +%s)
    - EXPIRATION_IN_DAYS=$(echo "($EXPIRATION_TIMESTAMP - $NOW)/(3600*24)" | bc)
    - HAS_EXPIRED=$(echo "$NOW > $EXPIRATION_TIMESTAMP" | bc)
    - ABOUT_TO_EXPIRE=$(echo "$EXPIRATION_IN_DAYS < $WARN_IN_DAYS" | bc )
    - echo "License expiration $EXPIRATION (in $EXPIRATION_IN_DAYS days)"
    - if [ "1" = "$HAS_EXPIRED" ]; then echo "Your LICENSE expired and KrakenD Enterprise cannot start!"; exit 1; fi
    - if [ "1" = "$ABOUT_TO_EXPIRE" ]; then echo "Your LICENSE will expire really soon, lower the threeshold of this warning to continue"; exit 1; fi

# Example to check the configuration using flexible configuration
check_config:
  stage: test
  image: {{< product image >}}:2.5
  variables:
    FC_ENABLE: 1
    FC_PARTIALS: $CI_PROJECT_DIR/config/partials
    FC_SETTINGS: $CI_PROJECT_DIR/config/settings/prod
    FC_TEMPLATES: $CI_PROJECT_DIR/config/templates
    FC_OUT: /tmp/krakend.json
    KRAKEND_FILE: $CI_PROJECT_DIR/config/krakend.tmpl
    KRAKEND_AUDIT_IGNORE: $CI_PROJECT_DIR/.krakend_audit_ignore
  script:
    - echo "FC_ENABLE is set to $FC_ENABLE"
    - echo "Runner working on path $(pwd)"
    - krakend check -tdc $KRAKEND_FILE
    - krakend check --lint -c $FC_OUT
    - krakend audit -c $FC_OUT --ignore-file=$KRAKEND_AUDIT_IGNORE --severity CRITICAL,HIGH
    - echo "--------------------------------------------------"
    - echo "------ YOU ROCK! KrakenD config looks good! ------"
    - echo "--------------------------------------------------"
  needs:
    - job: check-license
      optional: true

# Create an immutable Docker image
docker-build:
  image: docker:cli
  stage: build
  services:
    - docker:dind
  variables:
    DOCKER_IMAGE_NAME: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  before_script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
  # All branches are tagged with $DOCKER_IMAGE_NAME (defaults to commit ref slug)
  # Default branch is also tagged with `latest`
  script:
    - docker build --pull -t "$DOCKER_IMAGE_NAME" .
    - docker push "$DOCKER_IMAGE_NAME"
    - |
      if [[ "$CI_COMMIT_BRANCH" == "$CI_DEFAULT_BRANCH" ]]; then
        docker tag "$DOCKER_IMAGE_NAME" "$CI_REGISTRY_IMAGE:latest"
        docker push "$CI_REGISTRY_IMAGE:latest"
      fi
  # Run this job in a branch where a Dockerfile exists
  rules:
    - if: $CI_COMMIT_BRANCH
      exists:
        - Dockerfile

```