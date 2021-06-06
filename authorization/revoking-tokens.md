---
lastmod: 2020-04-01
date: 2018-11-05
linktitle: Revoking tokens
title: Revoking valid tokens with a Bloom filter
description: How to invalidate JWT tokens using the Bloom filter
weight: 40
source: https://github.com/devopsfaith/bloomfilter
menu:
  community_current:
    parent: "060 Authentication & Authorization"
---
The API Gateway authorizes users that provide valid tokens according to your criteria, but at some point, you might want to change your mind and decide to **revoke JWT tokens that are still valid**.

When are you going to need this? Examples of situations where you might need to revoke perfectly legit tokens: 
As a user, I want to log me out of all my devices.
As an administrator, I want to kick out someone from the platform.
As a software releaser, my new backend version requires new fields in the tokens, and I want all my sessions renegotiated again, or all users of a specific app (Android, iOS, Web app, etc.) has to be invalidated.

## Storing blocked tokens using the bloom filter
KrakenD integrates the [bloom filter](https://github.com/devopsfaith/bloomfilter) component that allows you to store in an optimized way tokens to revoke on the subsequent requests. 

When you enable the bloom filter, it inspects the payload of incoming JWT tokens to check if any of the configured fields in `TokenKeys` contains a blocked value. And if a blocked is found, access is not permitted.

The bloom filter component brings the following functionalities:

- Hold blocked tokens in memory
- Propagate blocked elements through an RPC interface
- Check tokens and discard access on positives

### Bloom filter client
The communication with the bloom filter is RPC based. The component exposes a listening port of your choice to receive updates of the bloom filter (single or batch), but a client is needed to communicate with the component.

The bloom filter library includes a [client](https://github.com/devopsfaith/bloomfilter/tree/master/cmd/client) so you can send the updates. In addition, the KrakenD Playground project includes a sample [web page with a form and an RPC client](https://github.com/devopsfaith/krakend-playground/tree/master/jwt-revoker) that sends commands to the bloom filter and updates it.

### Bloom filter performance
The Bloom filter is ideal for supporting a massive rejection of tokens with very little memory consumption. For instance, **100 million tokens** of any size consume around 0.5GB RAM (with a rate of false positives of 1 in 999,925,224 tokens), and lookups complete in constant time (*k* number of hashes). These numbers are impossible to get with a key-value or a relational database.

The tokens are in-memory, directly in the rejecter interface, so the system is quick resolving the match.

## Configuration
The bloom filter lives at the `extra_config` in the root level of the configuration, using the namespace `github_com/devopsfaith/bloomfilter`:


    "version": "2",
    "name": "My lovely gateway",
    "extra_config":{
      "github_com/devopsfaith/bloomfilter": {
        "N": 10000000,
        "P": 0.0000001,
        "HashName": "optimal",
        "TTL": 1500,
        "port": 1234,
        "TokenKeys": ["jti"]
      }
    }

All the configuration fields **are mandatory**, and are explained below:

- `N`: The maximum number of elements that you want to keep in memory. 
- `P`: The probability of returning a false positive.
- `HashName`: Either `optimal` (recommended) or `default`.
- `TTL`: The lifespan of the JWT you are generating, in seconds. The value must match the expiration you are setting in the - backend.
- `port`: The port number exposed (has to be free) for the RPC service to communicate with the bloomfilter.
- `TokenKeys`: The list with all the fields in your JWT payload that need watching. These fields establish the criteria to revoke accesses in the future.

The values `N` and `P` determine the size of the resulting bloom filter to fulfill your expectations. You can use this [bloom filter calculator](https://hur.st/bloomfilter/?n=1000000&p=1.0E-9&m=&k=) to play with the numbers.

{{< note title="Hygiene habits" >}}
Keep the life of your tokens short (e.g., 30 minutes).
{{< /note >}}

### Applied example
Our sample JWT payload has the following characteristics:

    "aud": "https://www.krakend.io",   
    "iss": "https://api.krakend.io",   
    "sub": "john@domain.com",  
    "jti": "mnb23vcsrt756yuiomnbvcx98ertyuiop",  
    "roles": ["user", "premium"],
    "did": "Android 8.0.0" 
    "exp": 1735689600

The following list shows the possible functionalities with an example`"TokenKeys": ["jti","sub","did","aud"]`:

- `jti` to revoke a single user session and device
- `sub` to revoke all sessions of the same subject.
- `did` to revoke all sessions using the same device ID (e.g., a new release in the Play Store)
- `aud` to revoke all our users of this audience or application.

Options are endless; these are some random examples, but it's up to you to decide which are the JWT elements you want to watch and apply revocations. If for instance, you only want to revoke access to a particular user or session, you only need to look at the `jti` (the unique identifier of a user) and `sub`.

## Expiring tokens in a cluster
All KrakenD nodes are stateless and act individually; they don't synchronize. Every node needs to receive the RPC notification about any tokens that need insertion in every local bloom filter.

The bloom filter gets updated while the service is running, but the level of synchronization between the nodes depends on your push strategy to the different members of the cluster. KrakenD uses conflict-free replicated data types (CRDT) so you can replicate the data across multiple computers in a network without coordination between the replicas, and where it is always mathematically possible to resolve inconsistencies which might result.

The resulting system is **eventually consistent**.

The bloom filter management is brought to you by the component, and for the administration part the client offers the necessary tools to adapt the gateway to your scenario. The implementation very much depends on what you want to achieve.

### Additional resources
If you want to learn bloomfilters by example, have a look at the following resources:

- [Bloomfilter tutorial](https://llimllib.github.io/bloomfilter-tutorial/)
- [Bloomfilter calculator](https://hur.st/bloomfilter/?n=1000000&p=1.0E-9&m=&k=)
