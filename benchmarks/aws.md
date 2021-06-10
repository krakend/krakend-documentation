---
lastmod: 2016-10-28
date: 2016-10-28
linktitle: On Amazon Web Services
title: KrakenD Benchmarks on AWS
description: Performance tests of KrakenD on Amazon AWS environment
weight: 10
menu:
  community_current:
    parent: "160 Benchmarks"
---

The following numbers show the execution results for the KrakenD benchmarks on [Amazon EC2](https://aws.amazon.com/ec2/) machines.

## Benchmark Setup
This set of benchmarks have been running on different AWS EC2 instances. Each individual test consists of spinning up 3 different machines, being:

- **A web server**: A [LWAN](https://lwan.ws/) web server using an instance `c4.xlarge`. This is the "fake API" where KrakenD will take the data
- **The HTTP load generator**: The machine actually running the load test. Uses **[hey](https://github.com/rakyll/hey)**, and runs in a `t2.medium`.
- **KrakenD**: Each different test uses a different instance type in Amazon:

The test consists in running `hey` against a KrakenD endpoint. The KrakenD endpoint uses as the backend an URL in (`LWAN`).
After running the test, the `hey` output is [parsed and converted to CSV](https://github.com/devopsfaith/hey-to-csv) in order to generate the graphs.

For each instance type there are 2 different tests:

- **Proxy**: When the KrakenD is just used as a gateway and calls to a single endpoint to the web server (`/foo` endpoint in the configuration).
- **Aggregate**: When the KrakenD calls to 3 different endpoints in the web server and aggregates the results (`/social` endpoint in the configuration).

The instance types we tested are:


| Instance Type | Number of vCPU | Memory |
|---------------|----|-------|
| t2.micro | 1 | 1 GB |
| t2.medium | 2 | 4 GB|
| m4.large | 2 | 8 GB|
| c4.xlarge | 4 | 7.5 GB|
| c4.2xlarge | 8 | 15 GB|


## KrakenD Configuration for all tests

The configuration for the load test was stored in the `krakend.json` file, as follows:

    {
      "version": 1,
      "host": [
        "http://lwan:8080"
      ],
      "endpoints": [
        {
          "endpoint": "/foo",
          "method": "GET",
          "backend": [
            {
              "url_pattern": "/bar"
            }
          ],
          "concurrent_calls": "1",
          "max_rate": 100000
        },
        {
          "endpoint": "/social",
          "method": "GET",
          "backend": [
            {
              "url_pattern": "/fb",
              "group": "fb"
            },
            {
              "url_pattern": "/youtube",
              "target": "data",
              "group": "youtube"
            },
            {
              "url_pattern": "/twitter",
              "group": "twitter"
            }
          ],
          "concurrent_calls": "1",
          "timeout": "500ms",
          "cache_ttl": "12h"
        }
      ],
      "oauth": {
        "disable": true
      },
      "cache_ttl": "5m",
      "timeout": "5s"
    }

Notice that `Lwan` is the backend running at `lwan:8080`.

And we started the KrakenD with this cmd (debug mode):
{{< terminal >}}
krakend run --config krakend.json -d > /dev/null
{{< /terminal >}}

## Results

### Proxy test on `t2.micro`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "t2_micro_test01.csv" >}}

### Aggregate test on `t2.micro`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "t2_micro_aggregate.csv" >}}

### Proxy test on `t2.medium`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "t2_medium_test01.csv" >}}

### Aggregate test on `t2.medium`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "t2_medium_aggregate.csv" >}}

### Proxy test on `m4.large`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "m4_large_test01.csv" >}}

### Aggregate test on `m4.large`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "m4_large_aggregate.csv" >}}

### Proxy test on `c4.xlarge`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "c4_xlarge_test01.csv" >}}

### Aggregate test on `c4.xlarge`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "c4_xlarge_aggregate.csv" >}}

### Proxy test on `c4.2xlarge`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "c4_2xlarge_test01.csv" >}}

### Aggregate test on `c4.2xlarge`

{{< gist kpacha 91caba50e47160f656069373b0f0605d "c4_2xlarge_aggregate.csv" >}}

### Conclusions
During all the tests we did, the instances of type `c4` always showed a stable behavior while the `m4` types didn't offer
a proportional increase in the performance and the variance of the responses is too high.

The instances `micro` provide nice figures of rps and latency for a good money. It looks like they suffer a little bit
more in the aggregated tests but in general it is a good choice.

To be taken into account that this type of service is CPU intensive so when using `t2` instances once you spend your CPU
credit the instance will perform worst.

**In general terms**:

- Use `micro` instances by default.
- If you expect high and continued load with complex use cases (intensive aggregation and manipulation) `c4.2xlarge` is worth it
- If you want to maintain quality of service with high load but a relative simple app, `c4.xlarge`
- For low to moderate loads use `micro` or a cluster of micros.
- We wouldn't choose `m4` in any scenario for the money/performance.

Look at the numbers and the use case you'll have in order to choose the right solution for you. And more importantly, do the tests
using your own data. This is a reference to contrast your own tests.
