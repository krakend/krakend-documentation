---
lastmod: 2022-01-28
date: 2022-01-28
linktitle: Understanding Spike Arrest and Burst
title: Understanding Spike Arrest and Burst
description: Spike Arrest throttling in KrakenD to regulate API traffic and prevent overload situations effectively
weight: 9300
notoc: true
images:
- /images/documentation/krakend-token-bucket.png
skip_header_image: true
menu:
  community_current:
    parent: "090 Traffic Management"
---
The **Spike Arrest** policy ensures a minimum time between different requests. KrakenD will enable Spike Arrest **after exhausting the burst capacity** of the rate-limiting features.

### Bursting control

The bursting control is the policy that defines what to do when you reach the throttling capacity of the system within a second.

When users consume content with rate-limiting enabled, the `capacity` of the rate limit defines the **bursting** they can have. Bursting makes users have a relatively higher number of requests for a short time. When this burst is exhausted, no additional requests are processed until the [Token Bucket algorithm](/docs/throttling/token-bucket/) credits the user again.

![Token Bucket image](/images/documentation/krakend-token-bucket.png)

The bursting control is automatically set on [endpoint rate limiting](/docs/endpoints/rate-limit/) with a capacity equal to the rate limit, and is configurable on [backend rate limit](/docs/backends/rate-limit/).

### Spike Arrest

The Spike Arrest policy defines the quickest time between two sequential requests when the users consume the maximum capacity.

After an emptied bucket (capacity exhausted), the following requests are in Spike Arrest mode and will need to have a delay of at least `1 ÷ max_rate` to be processed again. Krakend will reject connections requesting content faster than this rate.

Depending on the rate limit you implement, you might see rejected connections with status codes `503 Service Unavailable` or `429 Too Many Requests`.
