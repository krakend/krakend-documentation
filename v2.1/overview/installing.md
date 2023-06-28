---
lastmod: 2019-03-11
old_version: true
date: 2016-07-01
menu:
  community_v2.1:
    parent: "000 Getting Started"
title: KrakenD API Gateway Installation Guide
description: Follow our step-by-step installation guide to set up and configure KrakenD API Gateway, enabling efficient and scalable API management.
weight: 20
---
KrakenD is a **single binary file** that does not require any external libraries to work. To install KrakenD choose your operative system in the downloads section or use the Docker image.


{{< button-group >}}
{{< button url="/download/" text="Download" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9a9 9 0 01-9-9m9 9c1.657 0 3-4.03 3-9s-1.343-9-3-9m0 18c-1.657 0-3-4.03-3-9s1.343-9 3-9m-9 9a9 9 0 019-9" />
</svg>{{< /button >}}
{{< button url="https://designer.krakend.io/" type="inversed" >}}Generate a config file{{< /button >}}
{{< /button-group >}}

## Installation
You can install (or just run) KrakenD using different options.
{{< note title="Just exploring?" type="question" >}}
Use the [KrakenD Playground](https://github.com/krakendio/playground-community) if you want to play with KrakenD without configuring it. The Playground comes with several flavors of KrakenD and a mock API. Everything is ready to start playing, just do a `docker-compose up`!
{{< /note >}}

### Docker
The easiest way to get started is by pulling and running the [KrakenD image](https://hub.docker.com/r/devopsfaith/krakend/) from the Docker Hub.
{{< terminal title="Running KrakenD using the Docker container" >}}
docker run -p 8080:8080 -v $PWD:/etc/krakend/ devopsfaith/krakend run --config /etc/krakend/krakend.json
{{< /terminal >}}

If you choose not to mount the volume (the `-v`), a default `krakend.json` serving a `/__health` endpoint will be used. The volume expects to find a `krakend.json` in the current directory ([generate your first here](https://designer.krakend.io/)).

### AWS and Azure VM
See the [downloads page](/download/) for pre-built virtual machines in Azure and AWS.

### Mac OS X
The [Homebrew](https://brew.sh/) formula will download the source code, build the binary, and link the binary for you. The installation might take a while.

{{< terminal title="Install on Mac via Brew" >}}
brew install krakend
{{< /terminal >}}

After the installation completes go to [Using KrakenD](/docs/v2.1/overview/run/)

### Linux

#### CentOS, Oracle Linux, and Redhat (yum)
The installation process requires following these steps:

1. Install the repo package
2. Install the KrakenD package
3. Start the KrakenD service

Paste this in the terminal:
{{< terminal title="Yum based" >}}
rpm -Uvh {{< product download_repo >}}/rpm/{{< product rpm_source >}}
yum install -y krakend
systemctl start krakend
{{< /terminal >}}

#### Fedora
Paste this in the terminal:
{{< terminal title="DNF based" >}}
rpm -Uvh {{< product download_repo >}}/rpm/{{< product rpm_source >}}
dnf install -y krakend
systemctl start krakend
{{< /terminal >}}

The current KrakenD version will run at least in Centos 7 and Fedora 24

#### Debian and Ubuntu

The installation process requires following these steps:

1. Add the key
2. Add the repo to the sources.list
3. Update your package list
4. Install the KrakenD service

Bottom line:
{{< terminal title="DEB based" >}}
apt-key adv --keyserver keyserver.ubuntu.com --recv {{< param pgp_key >}}
echo "deb {{< product download_repo >}}/apt stable main" | tee /etc/apt/sources.list.d/krakend.list
apt-get update
apt-get install -y krakend
{{< /terminal >}}

Minimum supported versions are Debian 8, and Ubuntu 16.x.

#### Generic Linux (`tar.gz`)
You can also [download](/download/) the `tar.gz` and decompress it anywhere. Instructions to check the SHA and PGP signature [here](/docs/v2.1/overview/verifying-packages/).


## Compile from source
As KrakenD is open source you can opt for [building the binary](https://github.com/krakendio/krakend-ce). The binary you will produce is the same you can get in our download page, only that compiling it yourself always feels good!
