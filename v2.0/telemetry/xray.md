---
lastmod: 2019-09-15
old_version: true
date: 2019-09-15
notoc: true
linktitle: AWS X-Ray
title: Exporting traces to AWS X-Ray
weight: 110
notoc: true
menu:
  community_v2.0:
    parent: "080 Telemetry and Analytics"
meta:
  source: https://github.com/krakend/krakend-opencensus
  namespace:
  - telemetry/opencensus
  log_prefix:
  - "[SERVICE: Opencensus]"
  scope:
  - service
---
[AWS X-Ray](https://aws.amazon.com/xray/) is a service offered by Amazon that provides an end-to-end view of requests as they travel through your application, and shows a map of your application’s underlying components.

The Opencensus exporter allows you export data to AWS X-Ray. Enabling it only requires you to add the `xray` exporter in the [opencensus module](/docs/v2.0/telemetry/opencensus/).

The following configuration snippet sends data to your X-Ray:

{{< highlight json >}}
{
  "extra_config": {
    "telemetry/opencensus": {
      "exporters": {
        "xray": {
          "version": "latest",
          "region": "eu-west-1",
          "use_env": false,
          "access_key_id": "myaccesskey",
          "secret_access_key": "mysecretkey"
        }
      }
    }
  }
}
{{< /highlight >}}

- `version` (*string*): The version of the running application that is reporting data. Defaults to `KrakenD-opencensus`.
- `region` (*string*): The AWS geographical region.
- `use_env` (*boolean*): When `true` the AWS credentials (`access_key_id` and `secret_access_key`) are taken from environment vars. Don't specify them then.
- `access_key_id` (*string*): Your access key ID provided by Amazon. Needed when `use_env` is unset or set to `false`.
- `secret_access_key` (*string*): Your secret access key provided by Amazon. Needed when `use_env` is unset or set to `false`.


See also the [additional settings](/docs/v2.0/telemetry/opencensus/) of the Opencensus module that can be declared.
