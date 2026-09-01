---
lastmod: 2026-09-01
date: 2018-11-05
linktitle: Revoking tokens
title: Token Revocation
description: Learn how to implement token revocation mechanisms in KrakenD API Gateway to manage and invalidate access tokens when needed
weight: 40
source: https://github.com/krakend/bloomfilter
menu:
  community_current:
    parent: "080 Authentication & Authorization"
meta:
  namespace:
  - auth/revoker
  log_prefix:
  - "[SERVICE: Revoker Agent]"
---
The API Gateway authorizes users that provide valid tokens according to your criteria, but at some point, you might want to change your mind and decide to **revoke JWT tokens that are still valid**.

{{< note title="Revoke tokens via API" type="info" >}}
The Enterprise version offers a [Revoke Server](/docs/enterprise/authentication/revoke-server/) that coordinates token revokes in a cluster using a REST API.
{{< /note >}}


When are you going to need this? Examples of situations where you might need to revoke perfectly legit tokens:

- A user wants to log out from all my devices.
- An administrator wants to kick out someone from the platform.
- A software release needs all sessions renegotiated again, or users of a specific app (Android, iOS, Web app, etc.) have to be invalidated.

## Storing blocked tokens using the bloom filter
KrakenD integrates the [bloom filter](https://github.com/krakend/bloomfilter) component that allows you to store in an optimized way tokens to revoke on subsequent requests.

When you enable the bloom filter, it inspects the payload of incoming JWT tokens to check if any configured fields in `token_keys` contain a blocked value. And if a block is found, access is not permitted.

The bloom filter component brings the following functionalities:

- Hold blocked tokens in memory
- Propagate blocked elements through an RPC interface
- Check tokens and discard access on positives

### Bloom filter client
The communication with the bloom filter is RPC-based. The component exposes a listening port of your choice to receive updates of the bloom filter (single or batch), but **a client is needed to communicate with the component**.

**When using the open-source edition**, you have to build your client. Look at the bloom filter library, which includes a [client](https://github.com/krakend/bloomfilter/tree/master/cmd/client). In addition, the KrakenD Playground project consists of a sample [web page with a form and an RPC client](https://github.com/krakend/playground-community/tree/master/images/jwt-revoker) that sends commands to the bloom filter and updates it.

Note that this low-level bloom filter client requires elements added to the bloom filter to conform to a special format: a key, representing a field in the token, separated by a hypen (`-`); and the value of that field that will be used to revoke requests. [In the example below](#applied-example), you could expire the token for an individual user by adding `jti-mnb23vcsrt756yuiomnbvcx98ertyuiop` to the bloom filter. This can also be seen in the sample [web page with a form and an RPC client](https://github.com/krakend/playground-community/tree/master/images/jwt-revoker) in the KrakenD Playground project.

**When using the Enterprise edition** the [Revoke Server](/docs/enterprise/authentication/revoke-server/) connects to all KrakenD instances as a client, and there's nothing you need to build to make it work.

### Bloom filter performance
The Bloom filter is ideal for supporting a massive rejection of tokens with very little memory consumption. For instance, **100 million tokens** of any size consume around 0.5GB RAM (with a rate of false positives of 1 in 999,925,224 tokens), and lookups resolve in constant time (*k*-number of hashes). These numbers are impossible to get with a key value or a relational database.

The tokens are in-memory and directly in the rejecter interface, so the system quickly resolves the match.

## Securing the bloom filter RPC port
The `port` declared in the `auth/revoker` configuration opens an **RPC service on every KrakenD instance**, and that is the channel used to add elements to the bloom filter. This interface is **unauthenticated and unencrypted**: there is no API key, no TLS, and no authorization check. Any host able to open a TCP connection to that port can add revocations and query them.

Writing to the filter does require speaking its protocol: the RPC service uses Go's `net/rpc` with `gob` encoding, so a client has to be written in Go, either the [one included in the library](https://github.com/krakend/bloomfilter/tree/master/cmd/client) or an equivalent implementation. Keep in mind that this is a **requirement, not a protection**. It makes writing to the filter inconvenient, but it does not stop anyone who wants to do it, and it must never be counted as a security control.

Securing this port is therefore **your responsibility**, and the rule is simple: it must never be reachable from the outside. Treat it as an internal control plane, the same way you treat a database port.

### What an exposed RPC port allows
- **Locking out your users**. Anyone reaching the port can revoke any value of any watched claim. Depending on your `token_keys`, a single call invalidates one session (`jti`) or every token issued for an application (`aud`).
- **Damage that lasts until a restart**. Bloom filters do not support deletion, so an unwanted insertion cannot be undone. Recovering means restarting the affected instances, which also drops all the legitimate revocations they had.
- **Degrading the filter**. Writing junk consumes the capacity declared in `N`. Once you exceed it, the actual rate of false positives grows above the `P` you configured, and valid tokens start being rejected at random.
- **Probing revocations**. The interface also answers checks, so an unauthorized client can test whether a given claim value has been revoked.

### Recommendations
- **Do not publish the port**. Do not map it with `docker -p`, and do not add it to a Kubernetes `Service`, an Ingress, or a load balancer target group. The public listener of the gateway and the RPC port belong to different networks.
- **Filter at the network level**. The RPC service listens on all the interfaces of the machine and cannot be bound to a specific address from the configuration, so the restriction has to happen outside KrakenD: security groups, host firewall rules (`iptables`/`nftables`), or a Kubernetes `NetworkPolicy` that allows ingress to that port only from the hosts or pods running your revocation client.
- **Encrypt the traffic when it leaves the trusted network**. The protocol has no TLS. If the client and the gateways are not in the same private subnet, carry the traffic over a VPN, an IPsec/WireGuard tunnel, or a service mesh doing mTLS.
- **Protect the client as well**. Whatever pushes revocations (your own RPC client, an internal admin panel, or the Enterprise [Revoke Server](/docs/enterprise/authentication/revoke-server/)) becomes the entry point to the revocation mechanism and should be treated as an administrative service: authenticated, audited, and not publicly reachable.
- **Watch only the claims you need**. Every entry in `token_keys` widens the blast radius of an unauthorized write. If revoking individual sessions is enough for your use case, list `jti` only instead of broad claims such as `aud`.
- **Monitor connections to the port**. Any connection that does not come from your revocation client is a red flag, and network-level logging is the only place where you will see it, as the component does not authenticate callers.

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

{{< schema data="auth/revoker.json" filter="N,P,hash_name,TTL,port,token_keys">}}

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
- [Revoke Server](/docs/enterprise/authentication/revoke-server/) {{< badge>}}Enterprise{{< /badge >}}
