---
lastmod: 2018-11-08
date: 2018-11-05
linktitle: Revoking tokens
title: Revoking valid tokens
weight: 40
menu:
  documentation:
    parent: authorization
---
The API Gateway authorizes users that provide valid tokens according to your criteria, but at some point, you might want to change your mind and decide to **revoke tokens that are supposed to be still valid**.

Examples of revoking a valid token could be that an administrator of your application decides that a user can't longer login into the system, or you want to force the renegotiation of the tokens for a subset or all the users because you are not going to wait for the expiration.

KrakenD integrates the [bloomfilter](https://github.com/devopsfaith/bloomfilter) component that allows you to store in an optimized way tokens to revoke on the subsequent requests. When the bloomfilter is activated, the tokens are checked against the bloomfilter as if it were a blacklist, and if the token of the user matches de bloomfilter, the access is not permitted.

The bloomfilter component brings the following functionalities:

- Hold blacklisted tokens in memory
- Manage tokens through an RPC interface, both individually or in batch.
- Check tokens and discard access on positives

# Why bloomfilters?
The Bloomfilters are ideal for supporting massive rejection of tokens with very little memory consumption. For instance, 100 million tokens of any size consume around 0.5GB RAM (with a rate of false positives of 1 in 999,925,224 tokens). These numbers are impossible to get with a key-value or a relational database.

The tokens are in-memory, directly in the rejecter interface, so the system is quick resolving the match.

## Additional resources
If you want to learn bloomfilters by example, have a look at the following resources:

- [bloomfilter tutorial](https://llimllib.github.io/bloomfilter-tutorial/).
- [Bloomfilter calculator](https://hur.st/bloomfilter/?n=1000000&p=1.0E-9&m=&k=)

# Expiring tokens in a cluster
All KrakenD nodes are stateless and act individually. Every node needs to receive the RPC notification about any tokens that need insertion in every local bloomfilter.  The bloomfilter gets updated while the service is running, but it depends on how you push them in the cluster the synchronization differents. The system is **eventually consistent**.

The bloomfilter management is brought to you by the component, but you still need to implement the administration part that serves as the source of these tokens and the pushing of them (e.g., a queue and a consumer).