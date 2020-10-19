---
aliases:
- /overview/installing/
lastmod: 2019-03-11
date: 2016-07-01
menu:
  documentation:
    parent: getting started
title: Installing KrakenD
weight: 20
---
KrakenD is a **single binary file** that does not require any external libraries to work. To install KrakenD choose your operative system in the downloads section or use the Docker image.


<a href="/download" class="btn btn-secondary"><i class="fa fa-download"></i> Download KrakenD</a>
and
<a href="http://designer.krakend.io" class="btn btn-secondary"><i class="fa fa-file"></i> Generate the configuration file</a>


**Just exploring?**

Use the [KrakenD Playground](https://github.com/devopsfaith/krakend-playground) if you want to play with KrakenD without configuring it. The Playground comes with several flavors of KrakenD and a mock API. Everything is ready to start playing, just do a `docker-compose up`!

## Docker
If you are already familiar with Docker, the easiest way to get started is by pulling and running the [KrakenD image](https://hub.docker.com/r/devopsfaith/krakend/) from the Docker Hub.
{{< terminal title="Running KrakenD using the Docker container" >}}
docker run -p 8080:8080 -v $PWD:/etc/krakend/ devopsfaith/krakend run --config /etc/krakend/krakend.json
{{< /terminal >}}
Make sure you have a `krakend.json` in the current directory with your endpoint definition. You can [generate it here](http://designer.krakend.io/)

## Mac OS X
The [Homebrew](https://brew.sh/) formula will download the source code, build the formula and link the binary for you. The installation might take a while.

{{< terminal title="Install on Mac via Brew" >}}
brew install krakend
{{< /terminal >}}

After the installation completes go to [Using KrakenD](/docs/overview/usage/)

## Linux

### CentOS and Redhat
The installation process requires following these steps:

1. Install the repo package
2. Install the KrakenD package
3. Start the KrakenD service

Paste this in the terminal:
{{< terminal title="Yum based" >}}
rpm -Uvh {{< param download_repo >}}/rpm/krakend-repo-0.2-0.x86_64.rpm
yum install -y krakend
systemctl start krakend
{{< /terminal >}}

### Fedora
Paste this in the terminal:
{{< terminal title="DNF based" >}}
rpm -Uvh {{< param download_repo >}}/rpm/krakend-repo-0.2-0.x86_64.rpm
dnf install -y krakend
systemctl start krakend
{{< /terminal >}}

The current KrakenD version will run at least in Centos 7 and Fedora 24

### Debian and Ubuntu

The installation process requires following these steps:

1. Add the key
2. Add the repo to the sources.list
3. Update your package list
4. Install the KrakenD service

Bottom line:
{{< terminal title="DEB based" >}}
apt-key adv --keyserver keyserver.ubuntu.com --recv {{< param pgp_key >}}
echo "deb {{< param download_repo >}}/apt stable main" | tee /etc/apt/sources.list.d/krakend.list
apt-get update
apt-get install -y krakend
{{< /terminal >}}

The current KrakenD version will run at least in Debian 8, Debian 9 and Ubuntu 16.x

### Generic (via `tar.gz`)
You can also [download](/download) the `tar.gz` and decompress it anywhere. Instructions to check the SHA and PGP signature [here](/docs/overview/verifying-packages).


## Build KrakenD yourself
As KrakenD is open source you can opt for [building the binary](https://github.com/devopsfaith/krakend-ce). The binary you will produce is the same you can get in our download page, only that compiling it yourself always feels good!
