---
lastmod: 2016-06-23
date: 2016-07-01
title: Verifying packages (PGP and SHA256)
notoc: true
---
How to make sure what you are downloading is legit.

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

