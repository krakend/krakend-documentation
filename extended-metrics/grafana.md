---
lastmod: 2020-11-17
date: 2020-11-17
linktitle: Grafana dashboard
title: Preconfigured Grafana dashboard
weight: 30
notoc: true
menu:
  documentation:
    parent: extended-metrics
images:
- /images/documentation/grafana-screenshot.png
---

Adding a Grafana dashboard to KrakenD is a straightforward process:

1. Enable the [`/__stats/` endpoint](/docs/extended-metrics/stats-endpoint/) to let KrakenD expose its extended metrics
2. Import our [Grafana dashboard for Krakend](https://grafana.com/dashboards/5722)

To import the dashboard: From the Grafana UI, click the + icon in the side menu, and then click Import. Choose import via Grafana.com and use the ID `5722`.

The dashboard looks like this (video with no sound):

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ik18Zlwyap8" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
