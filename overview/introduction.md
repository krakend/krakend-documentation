---
aliases:
- /overview/introduction/
lastmod: 2018-09-27
date: 2016-10-25
linktitle: Introduction
menu:
  documentation:
    parent: getting started
next: /overview/quickstart
title: Introduction to KrakenD
weight: -10000
---

# What is KrakenD?
KrakenD is a high-performance open source solution to create enterprise-grade API Gateways.

Its core functionality is to create an API that acts as an aggregator of many microservices into single endpoints, doing automatically the heavy-lifting for you (group, wrap, transform, shrink, protocol translation, etc) and needs no programming as it offers a declarative way to create the endpoints. It is also well structured and layered and open to extending its functionality using plug-and-play middleware developed by the community or in-house.

KrakenD focuses on being a pure API gateway and unlike others is not coupled to the HTTP transport layer and it has been in production in large Internet businesses in Europe since early 2017.

# Why an API Gateway?

Consumers of REST API content (especially in microservices) often query backend services that weren't coded for the UI implementation. This is, of course, a good practice, but the UI consumers need to do implementations that suffer a lot of complexity and burden with the sizes of their microservices responses.

KrakenD is an **API Gateway** and proxy generator that sits between the client and all the source servers, adding a new layer that removes all the complexity to the clients, providing them only the information that the UI needs. KrakenD acts as an **aggregator** of many sources into single endpoints and allows you to group, wrap, transform and shrink responses. Additionally, it supports a myriad of middleware and plugins that allow you to extend the functionality, such as adding OAuth authorization or security layers (SSL, certificates, HTTP Strict Transport Security, Clickjacking protection, HTTP Public Key Pinning, MIME-sniffing prevention, XSS protection).

KrakenD not only supports HTTP(S), but because it is a set of generic libraries you can build all type of API Gateways and proxies, including, for instance, an RPC gateway.

KrakenD is written in [Go](https://golang.org/) with support for multiple platforms.

## Practical Example
A mobile developer needs to construct a single front page that requires data from several calls to their backend services, e.g:

    1) api.store.server/products
    2) api.store.server/marketing-promos
    3) api.users.server/users/{id_user}
    4) api.users.server/shopping-cart/{id_user}

The screen is very simple and _only_ needs to retrieve data from 4 different sources, wait for the round trip and then pick only a few fields of the response. Instead of thing these calls, the mobile could call a single endpoint to KrakenD:

    1) krakend.server/frontpage/{id_user}

And this is how it would look like:

![Gateway](/images/documentation/krakend-gateway.png)

The difference in size in this example would be because KrakenD server would have removed unneeded attributes from the responses.
