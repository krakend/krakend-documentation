---
lastmod: 2016-10-20
date: 2016-07-01
linktitle: Throttling Overview
title: KrakenD Throttling
weight: 1
menu:
  main:
    parent: Throttling and Limits
---

The KrakenD is a powerful tool that handles a huge amount of traffic and depending on the usage you could stress your
own backend micro-services architecture by requesting a lot of data, compromising your backend SLA.

In order to prevent the KrakenD to stress your infrastructure (or even someone using it to harm you) there are several
mechanisms to put you safe.


 - [The Circuit Breaker](/docs/throttling/circuit-breaker/)
 - [Rate limits](/docs/throttling/rate-limit/)
 - [Timeouts](/docs/throttling/timeouts/)
 - [Maximum IDLE connections](/docs/throttling/max-idle-connections/)
