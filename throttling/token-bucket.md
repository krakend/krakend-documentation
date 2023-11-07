---
lastmod: 2023-10-19
date: 2016-07-01
linktitle: Understanding the Token Bucket
title: "How API Traffic Throttling with Token Bucket algorithm works"
description: Implement token bucket-based throttling mechanism in KrakenD API Gateway to control API access rates and prevent abuse
weight: 10
notoc: true
images:
- /images/documentation/krakend-token-bucket.png
skip_header_image: true
menu:
  community_current:
    parent: "070 Traffic Management"
---

The Token Bucket algorithm helps you to allow or deny requests depending on the levels of traffic you are having. The algorithm is used to offer functionalities like the **Spike Arrest** and the several **Rate Limiting** options.

## A quick analogy

If you ever went to a traveling carnival, funfair, or amusement park to get into the attractions, you probably exchanged money for tokens/tickets at the ticket booth. The tokens help the carousel operator, bumper cars, or chance games stand to collect the payment faster and allow or deny access.

No matter how much money you have, you can carry a fixed maximum number of tokens in your pocket or a bucket before they spill. Picture yourself now with a bucket full of these. The number of rides and games you can have at the fair is the number of tokens left in the bucket (assuming 1 token = 1 ride). The bigger your bucket or pocket, the more rides you can do before going to the ticket booth to refill again.

Extrapolating to KrakenD, the rides are the API requests. Each token is a remaining gateway request you can do, and the bucket represents how many requests you are allowed to do before acquiring more tokens.

## Understanding the Token Bucket algorithm
The Token Bucket algorithm ([Wikipedia definition](https://en.wikipedia.org/wiki/Token_bucket)) is based on an analogy similar to the one described above.

KrakenD uses the bucket capacity to determine the number of requests it can serve. At the same time, it **fills the bucket with new tokens at a constant rate** while there is free space in it. Then, **users spend one token for each request**, and a token is removed from the bucket.

![Token Bucket algorithm](/images/documentation/krakend-token-bucket.png)

Users might spend the tokens faster than they are refilled. If the bucket gets empty, KrakenD rejects the requests until there is at least another token in the bucket.

We call `capacity` (or **max burst**) the number of tokens you can put in the bucket and `max_rate` the speed at which you refill the bucket. The `max_rate` determines the **maximum rate users will have on average**. The `capacity` and the `max_rate` can be different.

KrakenD adds a new token in the bucket every `1 ÷ max_rate` (for an `every` of `1s`). As each request is worth a token, you can serve as many requests as tokens remain in the bucket at every given time. The capacity (number of tokens) determines the maximum peak of requests you can absorb instantly. But remember that, on average, you can serve the number of requests in the `max_rate`.

**When the bucket is empty**, the [Spike Arrest](/docs/throttling/spike-arrest/) policy takes place and requests rejected.

### Example scenario
You want to limit users to **300 requests every minute**. A couple of ways to express this in the configuration could be:

- `max_rate=300` and `every=1m`, or
- `max_rate=5` and `every=1s` (300/60s=5)

When you define a `max_rate=5` with `every=1s` (or its alternative above), KrakenD will refill the bucket at a speed of one token every `1s ÷ 5 = 0.2s`. So, every 0.2 seconds, the bucket receives a new token. If your `every` uses units different than seconds, internally, they are converted to seconds for this calculation.

**On average** users can make five requests per second because if they push the system beyond the instant capacity, they need to wait 0.2 seconds to make another request, having the desired five every second. The `capacity` conditionates the effective number of requests you can do in a given instant as it holds the size of the bucket.

Suppose you have set a `capacity` of `10`. Then, a user could make ten requests under a second during a fresh start or while the bucket is full. Even though the `max_rate` is just five per second, the user can make more requests because the bucket is not empty after five (it has five more).

Has this user hacked your rate limit? No, you allowed the user to have a maximum burst of ten requests, but after the 10th request (emptied bucket), the gateway blocks the user and waits for the bucket to be refilled again at one token every 0.2 seconds.

When you do your tests, you have to be aware that the capacity can allow the user to exceed your planned maximum rate in specific cases, and as it happens with money, you can spend more than you earn (credit), but not forever.