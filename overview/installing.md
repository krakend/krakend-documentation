
---
aliases:
- /overview/installing/
lastmod: 2016-01-04
date: 2016-07-01
menu:
  main:
    parent: getting started
title: Installing KrakenD
weight: 20
---

# Linux and Mac OS X - from tar.gz
KrakenD is self-contained and does not require to have additional libraries in the system, so all you need
to do is run the binary. To install KrakenD go through the following steps:

1. Ensure the binary you have downloaded matches your architecture (check the [download](/download) section)
2. [Generate a `krakend.json`](http://designer.krakend.io/) configuration file using the [designer](http://designer.krakend.io/)
3. Start the KrakenD: `./krakend run --config krakend.json`

If you want this for production make sure to move `krakend` and `krakend.json` to a directory of your
choice, like `/etc/krakend/` or `/usr/local/krakend`.

If you attempt to run a binary in a platform it was not built for you might see a strange error. For
instance running krakend for Linux AMD64 in a Mac might show the following message:

    sh: exec format error: /Downloads/krakend/krakend_{{% version %}}_amd64

## CentOS and Redhat
The installation process requires following these steps:

1. Install the repo package
2. Install the KrakenD package
3. Start the KrakenD service

Paste this in the terminal:

    rpm -Uvh http://repo.krakend.io/rpm/krakend-repo-0.1-0.noarch.rpm
    yum install -y krakend
    systemctl start krakend

## Fedora
Paste this in the terminal:

    rpm -Uvh http://repo.krakend.io/rpm/krakend-repo-0.1-0.noarch.rpm
    dnf install -y krakend
    systemctl start krakend

Current KrakenD version will run at least in Centos 7 and Fedora 24

## Debian and Ubuntu

The installation process requires following these steps:

1. Add the key
2. Add the repo to the sources.list
3. Update your package list
4. Install the KrakenD service

Bottom line:

    apt-key adv --keyserver keyserver.ubuntu.com --recv AB39BEA1
    echo "deb http://repo.krakend.io/apt stable main" | tee /etc/apt/sources.list.d/krakend.list
    apt-get update
    apt-get install -y krakend

Current KrakenD version will run at least in Debian 8, Debian 9 and Ubuntu 16.x

# Docker
If you are already familiar with Docker the most convenient way to get started is by pulling the [KrakenD image](https://hub.docker.com/r/devopsfaith/krakend/) from Docker Hub:

    docker pull devopsfaith/krakend
    docker run -p 8080:8080 -v $PWD:/etc/krakend/ devopsfaith/krakend run --config /etc/krakend/krakend.json

Make sure you have a `krakend.json` in the current directory with your endpoint definition. You can [generate it here](http://designer.krakend.io/)



# PGP

We will check the detached signature [PGP](http://repo.krakend.io/bin/krakend_{{% version %}}_amd64.tar.gz.asc) against our package [KrakenD](http://repo.krakend.io/bin/krakend_{{% version %}}_amd64.tar.gz).

    $ gpg --verify krakend_{{% version %}}_amd64.tar.gz.asc krakend_{{% version %}}_amd64.tar.gz
    gpg: Signature made vie 02 dic 2016 19:07:49 CET using RSA key ID AB39BEA1
    gpg: Can't check signature: public key not found

We don't have the packager public key (AB39BEA1) in our system. You need to retrieve the public key from a key server.

    $ gpg --keyserver keyserver.ubuntu.com --recv-key AB39BEA1
    gpg: requesting key AB39BEA1 from hkp server keyserver.ubuntu.com
    gpg: trustdb created
    gpg: key AB39BEA1: public key "Daniel Ortiz <dortiz@devops.faith>" imported
    gpg: Total number processed: 1
    gpg:							 imported: 1	(RSA: 1)

Now you can verify the signature of the package:

    $ gpg --verify krakend_{{% version %}}_amd64.tar.gz.asc krakend_{{% version %}}_amd64.tar.gz
    gpg: Signature made jue 01 dic 2016 14:00:38 CET using RSA key ID AB39BEA1
    gpg: Good signature from "Daniel Ortiz <dortiz@devops.faith>"
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:					There is no indication that the signature belongs to the owner.
    Primary key fingerprint: EF9B 4CED 47D0 ECC9 69F8  3B4A 7377 C22B AB39 BEA1


# SHA256

To make sure the binary downloaded matches our SHA256 ensure the next 2 commands produce the same [SHA](http://repo.krakend.io/bin/krakend_{{% version %}}_amd64.tar.gz.sha256) output.

    # Your downloaded file
	$ shasum -a 256 -b krakend_{{% version %}}_amd64.tar.gz
    # Our SHA256
    $ curl http://repo.krakend.io/bin/krakend_{{% version %}}_amd64.tar.gz.sha256

