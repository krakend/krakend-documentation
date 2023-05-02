---
lastmod: 2021-10-18
old_version: true
date: 2018-11-05
linktitle: Revoking tokens
title: Revoking valid tokens with a Bloom filter
description: Invalidate valid JWT tokens individually or in groups according to your criteria with the bloom filter component
weight: 40
source: https://github.com/krakendio/bloomfilter
menu:
  community_v2.2:
    parent: "060 Authentication & Authorization"
meta:
  namespace:
  - auth/revoker
  log_prefix:
  - "[SERVICE: Revoker Agent]"
---
The API Gateway authorizes users that provide valid tokens according to your criteria, but at some point, you might want to change your mind and decide to **revoke JWT tokens that are still valid**.

When are you going to need this? Examples of situations where you might need to revoke perfectly legit tokens:

- A user wants to log out from all my devices.
- An administrator wants to kick out someone from the platform.
- A software release needs all sessions renegotiated again, or users of a specific app (Android, iOS, Web app, etc.) have to be invalidated.

## Storing blocked tokens using the bloom filter
KrakenD integrates the [bloom filter](https://github.com/krakendio/bloomfilter) component that allows you to store in an optimized way tokens to revoke on subsequent requests.

When you enable the bloom filter, it inspects the payload of incoming JWT tokens to check if any configured fields in `token_keys` contain a blocked value. And if a block is found, access is not permitted.

The bloom filter component brings the following functionalities:

- Hold blocked tokens in memory
- Propagate blocked elements through an RPC interface
- Check tokens and discard access on positives

### Bloom filter client
The communication with the bloom filter is RPC-based. The component exposes a listening port of your choice to receive updates of the bloom filter (single or batch), but **a client is needed to communicate with the component**.

**When using the open-source edition**, you have to build your client. Look at the bloom filter library, which includes a [client](https://github.com/krakendio/bloomfilter/tree/master/cmd/client). In addition, the KrakenD Playground project consists of a sample [web page with a form and an RPC client](https://github.com/krakendio/playground-community/tree/master/images/jwt-revoker) that sends commands to the bloom filter and updates it.

**When using the Enterprise edition** the [Revoke Server](/docs/enterprise/authentication/revoke-server/) connects to all KrakenD instances as a client, and there's nothing you need to build to make it work.

### Bloom filter performance
The Bloom filter is ideal for supporting a massive rejection of tokens with very little memory consumption. For instance, **100 million tokens** of any size consume around 0.5GB RAM (with a rate of false positives of 1 in 999,925,224 tokens), and lookups resolve in constant time (*k*-number of hashes). These numbers are impossible to get with a key value or a relational database.

The tokens are in-memory and directly in the rejecter interface, so the system quickly resolves the match.

## Configuration
The bloom filter lives at the `extra_config` in the root level of the configuration, using the namespace `auth/revoker`:

```json
{
    "version": "2",
    "name": "My lovely gateway",
    "extra_config":{
      "auth/revoker": {
        "N": 10000000,
        "P": 0.0000001,
        "hash_name": "optimal",
        "TTL": 1500,
        "port": 1234,
        "token_keys": ["jti"]
      }
    }
}
```



All the configuration fields **are mandatory** and are explained below:

{{< schema version="v2.2" data="auth/revoker.json" filter="N,P,hash_name,TTL,port,token_keys">}}

If you use the bloom filter together with the Revoken Server {{< badge >}}Enterprise{{< /badge >}}, see [its configuration](/docs/enterprise/authentication/revoke-server/).


{{< note title="Hygiene habits" >}}
Keep the life of your tokens short (e.g., 30 minutes).
{{< /note >}}

### Applied example
Our sample JWT payload has the following characteristics:

```json
{
    "aud": "https://www.krakend.io",
    "iss": "https://api.krakend.io",
    "sub": "john@domain.com",
    "jti": "mnb23vcsrt756yuiomnbvcx98ertyuiop",
    "roles": ["user", "premium"],
    "did": "Android 8.0.0",
    "exp": 1735689600
}
```


The following list shows the possible functionalities with an example`"token_keys": ["jti","sub","did","aud"]`:

- `jti` to revoke a single user session and device
- `sub` to revoke all sessions of the same subject.
- `did` to revoke all sessions using the same device ID (e.g., a new release in the Play Store)
- `aud` to revoke all our users of this audience or application.

Options are endless; these are some random examples, but it's up to you to decide which JWT elements you want to watch and apply revocations. If, for instance, you only want to revoke access to a particular user or session, you only need to look at the `jti` (the unique identifier of a user) and `sub`.

## Expiring tokens in a cluster
All KrakenD nodes are stateless and act individually; they don't synchronize. Every node must receive the RPC notification about any tokens that need insertion in every local bloom filter.

The bloom filter gets updated while the service is running, but the level of synchronization between the nodes depends on your push strategy to the different cluster members. KrakenD uses conflict-free replicated data types (CRDT), so you can replicate the data across multiple computers in a network without coordination between the replicas, and where it is always mathematically possible to resolve inconsistencies that might result.

The resulting system is **eventually consistent**.

The bloom filter management is brought to you by the component, and for the administration part, the client offers the necessary tools to adapt the gateway to your scenario. The implementation very much depends on what you want to achieve.

### Additional resources
If you want to learn bloomfilters by example or additional information on token revocation, have a look at the following resources:

- [Bloomfilter tutorial](https://llimllib.github.io/bloomfilter-tutorial/)
- [Bloomfilter calculator](https://hur.st/bloomfilter/?n=1000000&p=1.0E-9&m=&k=)
- [Revoke Server](/docs/enterprise/authentication/revoke-server/) {{< badge color="denim">}}Enterprise{{< /badge >}}
