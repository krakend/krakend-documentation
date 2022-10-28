---
lastmod: 2019-03-11
old_version: true
date: 2016-07-01
title: Verifying packages (PGP and SHA256)
notoc: true
---
How to make sure what you are downloading is legit.

## PGP
We will check the detached signature [PGP]({{< product download_repo >}}/bin/krakend_1.3.0_amd64.tar.gz.asc) against our package [KrakenD]({{< product download_repo >}}/bin/krakend_1.3.0_amd64.tar.gz).

    $ gpg --verify krakend_1.3.0_amd64.tar.gz.asc krakend_1.3.0_amd64.tar.gz
    gpg: Signature made Sun Mar 10 18:17:18 2019 UTC using RSA key ID {{< param pgp_key >}}
    gpg: Can't check signature: public key not found

We don't have the packager public key (AB39BEA1) in our system. You need to retrieve the public key from a key server.

    $ gpg --keyserver keyserver.ubuntu.com --recv-key {{< param pgp_key >}}
    gpg: requesting key {{< param pgp_key >}} from hkp server keyserver.ubuntu.com
    gpg: trustdb created
    gpg: key {{< param pgp_key >}}: public key "Devops Faith Package Manager <packages@devops.faith>" imported
    gpg: Total number processed: 1
    gpg:							 imported: 1	(RSA: 1)

Now you can verify the signature of the package:

    $ gpg --verify krakend_1.3.0_amd64.tar.gz.asc krakend_1.3.0_amd64.tar.gz
    gpg: Signature made Sun Mar 10 18:17:18 2019 UTC using RSA key ID {{< param pgp_key >}}
    gpg: Good signature from "Devops Faith Package Manager <packages@devops.faith>"
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:					There is no indication that the signature belongs to the owner.
    Primary key fingerprint: {{< param pgp_fingerprint >}}


## SHA256

To make sure the binary downloaded matches our SHA256 ensure the next 2 commands produce the same [SHA]({{< product download_repo >}}/bin/krakend_1.3.0_amd64.tar.gz.sha256) output.

    ## Your downloaded file
	$ shasum -a 256 -b krakend_1.3.0_amd64.tar.gz
    ## Our SHA256
    $ curl {{< product download_repo >}}/bin/krakend_1.3.0_amd64.tar.gz.sha256
