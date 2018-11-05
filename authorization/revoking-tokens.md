---
lastmod: 2018-11-05
date: 2018-11-05
linktitle: Revoking tokens
title: Revoking valid tokens
weight: 40
menu:
  documentation:
    parent: authorization
---
<span class="badge badge-warning">This document is a draft</span> (functionality is well tested though)

At some point in time you might want to decide to revoke tokens that are valid.

KrakenD integrates a [bloomfilter](https://github.com/devopsfaith/bloomfilter) that allows you to reject tokens that were issued correctly and are still unexpired.

The Bloomfilter is designed to support massive rejection of tokens with very little memory consumption.