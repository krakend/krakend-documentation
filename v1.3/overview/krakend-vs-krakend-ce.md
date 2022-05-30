---
lastmod: 2018-10-20
canonical: "/docs/overview/lura-vs-krakend/"
old_version: true
date: 2018-06-23
linktitle: Lura vs. KrakenD
description: What is the difference between Lura and KrakenD?
notoc: true
menu:
  community_v1.3:
    parent: "000 Getting Started"
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

{{< button-group >}}
{{< button url="https://luraproject.org" text="Website" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9a9 9 0 01-9-9m9 9c1.657 0 3-4.03 3-9s-1.343-9-3-9m0 18c-1.657 0-3-4.03-3-9s1.343-9 3-9m-9 9a9 9 0 019-9" />
</svg>{{< /button >}}
{{< button url="https://github.com/luraproject/lura" type="inversed" >}}Source code{{< /button >}}
{{< /button-group >}}

### KrakenD API gateway
`KrakenD` ([repo](https://github.com/devopsfaith/krakend-ce)) is our ready-to-use API gateway, assembled the way we think it delivers more value to the general audience. KrakenD uses the Lura Project in its core and extends its functionality by adding in the final binary multiple middlewares [contributions](https://github.com/krakendio/krakend-cobra) we thought an API Gateway should have.

KrakenD adds to Lura more functionality like logging, service discovery, developer tools, metrics, circuit breaker, rate limiting, OAuth, security, and other exciting stuff.

{{< button-group >}}
{{< button url="/download/" text="Download" >}}
<svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
<path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" />
</svg>{{< /button >}}
{{< button url="https://github.com/devopsfaith/krakend-ce" type="inversed" >}}Source code{{< /button >}}
{{< /button-group >}}

### KrakenD Enterprise
Customers of the [KrakenD Enterprise](/enterprise) package enjoy the development, consultancy, support, and training services offered by the very same KrakenD creators (and Lura steering committee). As per the software, KrakenD Enterprise users have SaaS functionalities that allow remote management, observability, and other features. There is also more tooling around KrakenD to increase productivity and enable working with KrakenD in large groups of developers.

**Our commitment to open-source is in the center of our business**, and this is why our Enterprise solution is built on top of the open-source version.  The Enterprise version uses the same OSS binary and extends it with a great variety of pluggable solutions. We want to make sure that both enterprise and community users have the excellent quality and reliability they are used to.

[Learn more about Enterprise](/enterprise/)
