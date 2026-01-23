---
lastmod: 2022-09-21
old_version: true
date: 2022-09-21
notoc: true
linktitle: Logging Dashboard (ELK)
title: ELK Integration for Logging
description: Learn how to integrate the ELK (Elasticsearch, Logstash, Kibana) stack for centralized logging and log analysis in KrakenD API Gateway
weight: 390
source: https://github.com/krakend/telemetry-dashboards
menu:
  community_v2.12:
    parent: "160 Monitoring, Logs, and Analytics"
images:
- /images/documentation/screenshots/kibana-krakend-dashboard.png
---
KrakenD can push logs to external services; a good example is an integration with the ELK Stack (**Elastic + Logstash + Kibana**). The ELK integration allows you to have KrakenD pushing logs to your Elastic server and **visualize them through a Kibana dashboard**.

The Kibana dashboard lets you monitor the logging activity of the gateway and identify problems quickly. The included dashboard is a starting point that provides typical graphs and metrics, but you can extend it as per your needs and add other metrics to watch.

## ELK Configuration
The configuration you need on your `krakend.json` to enable ELK integration is:

```json
{
  "$schema": "https://www.krakend.io/schema/v2.12/krakend.json",
  "version": 3,
  "extra_config": {
    "telemetry/logging": {
      "level": "DEBUG",
      "@comment": "Prefix should always be inside [] to keep the grok expression working",
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
```

There's nothing else on KrakenD that you need to do.

{{< note title="Where are the logs now?" type="info" >}}
When you enable the ELK integration, you will stop seeing the application logs on `stdout` as KrakenD pushes them to the ELK stack.
{{< /note >}}

## Logstash and Kibana configuration
The configuration files you need for Logstash and Kibana can be downloaded from the [Telemetry Dashboards repository](https://github.com/krakend/telemetry-dashboards).

{{< button-group >}}
{{< button url="https://github.com/krakend/telemetry-dashboards" type="inversed" text="Download ELK configuration files" >}}<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 496 512"><!--! Font Awesome Free 6.2.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license/free (Icons: CC BY 4.0, Fonts: SIL OFL 1.1, Code: MIT License) Copyright 2022 Fonticons, Inc. --><path d="M165.9 397.4c0 2-2.3 3.6-5.2 3.6-3.3.3-5.6-1.3-5.6-3.6 0-2 2.3-3.6 5.2-3.6 3-.3 5.6 1.3 5.6 3.6zm-31.1-4.5c-.7 2 1.3 4.3 4.3 4.9 2.6 1 5.6 0 6.2-2s-1.3-4.3-4.3-5.2c-2.6-.7-5.5.3-6.2 2.3zm44.2-1.7c-2.9.7-4.9 2.6-4.6 4.9.3 2 2.9 3.3 5.9 2.6 2.9-.7 4.9-2.6 4.6-4.6-.3-1.9-3-3.2-5.9-2.9zM244.8 8C106.1 8 0 113.3 0 252c0 110.9 69.8 205.8 169.5 239.2 12.8 2.3 17.3-5.6 17.3-12.1 0-6.2-.3-40.4-.3-61.4 0 0-70 15-84.7-29.8 0 0-11.4-29.1-27.8-36.6 0 0-22.9-15.7 1.6-15.4 0 0 24.9 2 38.6 25.8 21.9 38.6 58.6 27.5 72.9 20.9 2.3-16 8.8-27.1 16-33.7-55.9-6.2-112.3-14.3-112.3-110.5 0-27.5 7.6-41.3 23.6-58.9-2.6-6.5-11.1-33.3 2.6-67.9 20.9-6.5 69 27 69 27 20-5.6 41.5-8.5 62.8-8.5s42.8 2.9 62.8 8.5c0 0 48.1-33.6 69-27 13.7 34.7 5.2 61.4 2.6 67.9 16 17.7 25.8 31.5 25.8 58.9 0 96.5-58.9 104.2-114.8 110.5 9.2 7.9 17 22.9 17 46.4 0 33.7-.3 75.4-.3 83.6 0 6.5 4.6 14.4 17.3 12.1C428.2 457.8 496 362.9 496 252 496 113.3 383.5 8 244.8 8zM97.2 352.9c-1.3 1-1 3.3.7 5.2 1.6 1.6 3.9 2.3 5.2 1 1.3-1 1-3.3-.7-5.2-1.6-1.6-3.9-2.3-5.2-1zm-10.8-8.1c-.7 1.3.3 2.9 2.3 3.9 1.6 1 3.6.7 4.3-.7.7-1.3-.3-2.9-2.3-3.9-2-.6-3.6-.3-4.3.7zm32.4 35.6c-1.6 1.3-1 4.3 1.3 6.2 2.3 2.3 5.2 2.6 6.5 1 1.3-1.3.7-4.3-1.3-6.2-2.2-2.3-5.2-2.6-6.5-1zm-11.4-14.7c-1.6 1-1.6 3.6 0 5.9 1.6 2.3 4.3 3.3 5.6 2.3 1.6-1.3 1.6-3.9 0-6.2-1.4-2.3-4-3.3-5.6-2z"/></svg>
</svg>
{{< /button >}}
{{< /button-group >}}

### Logstash
The `logstash.conf` file includes an example of a Logstash configuration. First, change the **hostname** of your Elasticsearch server and any custom ports you might use. Then, start Logstash with this configuration to properly ingest KrakenD logs.

### Kibana
To import the Kibana dashboard included in the ELK repository above, execute the following command once your Kibana is up and running. Replace `localhost:5601` if needed:

{{< terminal title="Term" >}}
curl -X POST "localhost:5601/api/saved_objects/_import" -H "kbn-xsrf: true" --form file=@dashboard.ndjson -H "kbn-xsrf: true"
{{< /terminal >}}




## ELK live demo
If you want to see how this works, you can start the [KrakenD Playground](/docs/v2.12/overview/playground/).