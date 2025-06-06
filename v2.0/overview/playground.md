---
lastmod: 2018-09-27
old_version: true
date: 2018-09-13
linktitle: The Playground
description: Start playing with KrakenD through a ready-to-use complete environment -  gateway, backend, client and several use cases.
notoc: true
source: https://github.com/krakend/krakend-playground
menu:
  community_v2.0:
    parent: "000 Getting Started"
title: The KrakenD Playground
weight: 110
images: ["https://github.com/krakend/krakend-playground/blob/master/composer-env.png?raw=true"]

---
If you are new to KrakenD, a quick way to get started is to make use of the [KrakenD Playground](https://github.com/krakend/krakend-playground).

The KrakenD Playground is a **Docker Compose** environment that puts together the necessary pieces to let you play with KrakenD in a working environment.

As KrakenD is an API gateway, we have also added to the environment an API (backend) to feed the gateway and a website to make use of the data. With the KrakenD Playground, you can see the different pieces that take place in the API Gateway use cases.

## An environment ready to use!
The KrakenD Playground can be started with `docker compose up` and then fire up your browser, `curl`, Postman, httpie or anything else you use to interact with the services.

When starting the Docker compose you will have:

- The KrakenD API Gateway, on port 8080
- A client consuming data from the gateway, on port 3000
- A backend simulating API responses on port 8000
- Plus a [Jaeger](https://www.jaegertracing.io/) where traces are sent on port 16686

Have a look at the `docker-compose.yml` file to understand how these services interact together, and also at the `krakend.json` file to find out what endpoints are exposed.

If you'd like to see more examples, please let us know or open an [issue](https://github.com/krakend/krakend-playground/issues)!
