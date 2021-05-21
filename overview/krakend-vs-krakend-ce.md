---
lastmod: 2018-10-20
date: 2018-06-23
linktitle: Lura vs. KrakenD
description: What is the difference between Lura and KrakenD?
notoc: true
menu:
  documentation:
    parent: getting started
title: Lura vs. KrakenD
weight: 100
---
If you had a quick look at our git repositories or the documentation, you might be confused at first, as there is something called the **Lura Project** and also **KrakenD**.

## TL;DR; Difference between Lura, KrakenD, and Enterprise

- [**Lura**](https://luraproject.org) is the KrakenD's engine. Formerly known as "**KrakenD framework**" until we [donated it to The Linux Foundation on 2021](/blog/krakend-framework-joins-the-linux-foundation/). It is not a product itself but a set of libraries.
- **KrakenD** is our open-source API Gateway ready to use
- **KrakenD Enterprise** is our commercial version, including services to businesses

### Lura Project
The [Lura Project](https://luraproject.org) is our original *KrakenD framework* that we [donated to The Linux Foundation on 2021](/blog/krakend-framework-joins-the-linux-foundation/). Previously it lived from 2006 to 2021 under the KrakenD's team organization ([@krakend_io](https://twitter.com/krakend_io)) until it became a [Linux Foundation](https://linuxfoundation.org/) project. KrakenD team continues to be part of the steering committee.

The mission of the Lura Project is to offer an extendable, simple and stateless high-performance API Gateway **framework** designed for both cloud-native and on-prem setups. It provides components for assembling your API Gateway. It can be used in its entirety or just importing it as Go libraries to take only some of the functionality it brings. 

Lura focuses on providing the core functionality that a pure API gateway needs. It keeps it clean and extensible so that you can create your custom gateway without any trouble. KrakenD is the primary representation of Lura's possibilities.

<a class="btn btn-secondary btn-circle" href="https://luraproject.org"><i class="fa fa-globe"></i> Website</a>
<a class="btn btn-light" href="https://github.com/luraproject/lura"><i class="fab fa-github"></i> Source code</a>

### KrakenD API gateway
`KrakenD` ([repo](https://github.com/devopsfaith/krakend-ce)) is our ready-to-use API gateway, assembled the way we think it delivers more value to the general audience. KrakenD uses the Lura Project in its core and extends its functionality by adding in the final binary multiple middlewares [contributions](https://github.com/devopsfaith/krakend-contrib) we thought an API Gateway should have.

KrakenD adds to Lura more functionality like logging, service discovery, developer tools, metrics, circuit breaker, rate limiting, OAuth, security, and other exciting stuff.

<a class="btn btn-secondary btn-circle" href="/download/"><i class="fa fa-download"></i> Download</a>
<a class="btn btn-light" href="https://github.com/devopsfaith/krakend-ce"><i class="fab fa-github"></i> Source code</a>

### KrakenD Enterprise
Customers of the [KrakenD Enterprise](/enterprise) package enjoy the development, consultancy, support, and training services offered by the very same KrakenD creators (and Lura steering committee). As per the software, KrakenD Enterprise users have SaaS functionalities that allow remote management, observability, and other features. There is also more tooling around KrakenD to increase productivity and enable working with KrakenD in large groups of developers.

**Our commitment to open-source is in the center of our business**, and this is why our Enterprise solution is built on top of the open-source version.  The Enterprise version uses the same OSS binary and extends it with a great variety of pluggable solutions. We want to make sure that both enterprise and community users have the excellent quality and reliability they are used to.

[Learn more about Enterprise](/enterprise/)
