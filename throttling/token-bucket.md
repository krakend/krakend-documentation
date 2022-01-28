---
lastmod: 2018-09-29
date: 2016-07-01
linktitle: Token Bucket
title: Understanding the Token Bucket algorithm
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

## A quick analogy...

If you ever went to a travelling carnival, funfair, or amusement park, to get into the attractions, you probably exchanged money for tokens/tickets at the ticket booth. The tokens help the operator of the carousel, bumper cars, or chance games stand, to collect the payment faster and know who can jump in and who doesn't.

No matter how much money you have, there is a fixed maximum number of tokens you can carry in your pocket or in a bucket before they spill. Picture yourself now with a bucket full of these. The amount of rides and games you can have at the fair, is the number of tokens left you have in the bucket (assuming 1 token = 1 ride). The bigger is your bucket or pocket, the more rides you can do before needing to go to the ticket booth and wait for your turn again.

Extrapolating to KrakenD, the rides are the API requests. Each token is a remaining gateway request you can do, and the bucket represents how many requests you are allowed to do before you need to wait again.

## How the Token Bucket algorithm works
The Token Bucket algorithm ([Wikipedia](https://en.wikipedia.org/wiki/Token_bucket)) is based in an analogy similar to the one described above.

![Token Bucket algorithm](/images/documentation/krakend-token-bucket.png)

KrakenD uses the bucket capacity to determine the number of requests that can serve at once. At the same time, it **fills the bucket with new tokens at a constant rate** and while there is free space in it. Then, **users spend one token for each request** and it's removed from the bucket.

It might happen that users spend the tokens faster than they are refilled. If the bucket gets empty, the requests are rejected until KrakenD adds at least another token.

We call `capacity` (or **max burst**) to the number of tokens you can put in the bucket, and `max_rate` to the average throughput you allow. The `max_rate` determines the speed at which the tokens are added into the bucket. Depending on the rate limit type, the `capacity` and the `max_rate` can be different.

KrakenD adds a token in the bucket every `1 ÷ max_rate` seconds. As each connection is worth a token, you can serve at every given point in time as many concurrent requests as tokens are remaining in the bucket. The capacity (number of tokens) determines the maximum peak of requests you can absorve in an instant. But remember that in average, you will be able to serve the number of requests in the `max_rate`.

**When the bucket is empty**, the [Spike Arrest](/docs/throttling/spike-arrest/) policy takes place.

**Example:**

If you define a `max_rate=5` it means that in average users will be able to do 5 requests per second. The speed at which KrakenD refills the bucket with one token will be every `1 ÷ 5 = 0.2` seconds.