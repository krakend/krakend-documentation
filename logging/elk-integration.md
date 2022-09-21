---
lastmod: 2022-09-21
date: 2022-09-21
notoc: true
linktitle: ELK dashboard
title: ELK Stack dashboard
weight: 100
source: https://github.com/krakendio/elk-dashboard
menu:
  community_current:
    parent: "090 Logging"
images:
- /images/documentation/screenshots/kibana-krakend-dashboard.png
---
KrakenD can push logs to external services; a good example is an integration with the ELK Stack (Elastic + Logstash + Kibana). The ELK integration allows you to have KrakenD pushing logs to your Elastic server and **visualize them through a Kibana dashboard**.

The Kibana dashboard lets you monitor the logging activity of the gateway and identify problems quickly. The included dashboard is a starting point that provides typical graphs and metrics, but you can extend it as per your needs and add other metrics to watch.

## ELK Configuration
The configuration you need on your `krakend.json` to enable ELK integration is:

{{< highlight json >}}
{
  "$id": "https://www.krakend.io/schema/v3.json",
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "DEBUG",
      "@comment": "Prefix should always be inside [] to keep the grok expression working"
      "prefix": "[KRAKEND]",
      "syslog": false,
      "stdout": true
    },
    "telemetry/gelf": {
      "address": "logstash:12201",
      "enable_tcp": false
    }
  }
}
{{< /highlight >}}

There's nothing else on KrakenD that you need to do.

{{< note title="Where are the logs now?" type="info" >}}
When you enable the ELK integration, you will stop seeing the application logs on `stdout` as KrakenD pushes them to the ELK stack.
{{< /note >}}

## Logstash and Kibana configuration
The configuration files you need for Logstash and Kibana can be downloaded from the [ELK integration repository](https://github.com/krakendio/elk-integration).

Copy the configuration file `logstash.conf` to your logstash server and adjust the address as needed.

## ELK live demo
If you want to see how this works, you can start the [KrakenD Playground](/docs/overview/playground/).