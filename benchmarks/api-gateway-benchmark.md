---
lastmod: 2016-10-28
date: 2016-10-28
linktitle: KrakenD vs others
title: Comparison of KrakenD vs other products in the market (Benchmark)
description: Performance tests of KrakenD
weight: 10
menu:
  community_current:
    parent: "160 Benchmarks"
---
We wanted to compare our own product with other similar products in the market. In order to do so we used the same
environment and conditions and tested the following products:

 - Kong
 - Vulcand
 - Tyk
 - KrakenD

For the benchmarks, we based the tests on the benchmarking project [varnish/api-gateway-benchmarks](https://github.com/varnish/api-gateway-benchmarks).

**At the time of writing, KrakenD does not support auth features, so we just did the benchmark with _test01_**

## Hardware

    Model MacBook Pro (MacBookPro11,4) - August 2015
    Processor:    Intel Core i7 2,2 GHz

## Setup

For this test, we stored this configuration at `krakend.json`

    {
      "version": 1,
      "host": [
        "http://webserver:8888"
      ],
      "endpoints": [
        {
          "endpoint": "/test01",
          "method": "GET",
          "concurrent_calls": "1",
          "backend": [
            {
              "url_pattern": "/test01"
            }
          ]
        },
        {
          "endpoint": "/test03",
          "method": "GET",
          "backend": [
            {
              "url_pattern": "/test03"
            }
          ],
          "max_rate": "1000000"
        },
        {
          "endpoint": "/test04",
          "method": "GET",
          "max_rate": "1",
          "backend": [
            {
              "url_pattern": "/test04"
            }
          ]
        }
      ],
      "oauth": {
        "disable": true
      },
      "cache_ttl": "5m",
      "timeout": "5s"
    }

And we started the environment as described in the [README](https://github.com/varnish/api-gateway-benchmarks/blob/master/README.md#deployment-example)

    $ cd deployment/vagrant
    $ vagrant up

    $ vagrant ssh webserver
    [vagrant@webserver ~]$ cd /opt/benchmarks/webservers/dummy-api
    [vagrant@webserver dummy-api]$ sudo ./deploy
    [vagrant@webserver dummy-api]$ exit

    $ vagrant ssh gateway
    [vagrant@gateway ~]$ cd /opt/benchmarks/gateways/krakend
    [vagrant@gateway krakend]$ sudo ./deploy
    [vagrant@gateway krakend]$ exit

    $ vagrant ssh consumer
    [vagrant@consumer ~]$ cd /opt/benchmarks/consumers/boom
    [vagrant@consumer boom]$ sudo ./deploy
    [vagrant@consumer boom]$ /usr/local/bin/test00
    [vagrant@consumer boom]$ /usr/local/bin/test01

## Results

### Test 00

    [vagrant@consumer ~]$ /usr/local/bin/test00
    100000 / 100000 Booooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00 %

    Summary:
      Total:  13.2722 secs.
      Slowest:  0.1720 secs.
      Fastest:  0.0002 secs.
      Average:  0.0132 secs.
      Requests/sec: 7534.5524
      Total Data Received:  10800000 bytes.
      Response Size per Request:  108 bytes.

    Status code distribution:
      [200] 100000 responses

    Response time histogram:
      0.000 [1]     |
      0.017 [72298] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.035 [26760] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.052 [522]   |
      0.069 [227]   |
      0.086 [93]    |
      0.103 [28]    |
      0.120 [4]     |
      0.138 [10]    |
      0.155 [24]    |
      0.172 [33]    |

    Latency distribution:
      10% in 0.0039 secs.
      25% in 0.0076 secs.
      50% in 0.0129 secs.
      75% in 0.0180 secs.
      90% in 0.0218 secs.
      95% in 0.0233 secs.
      99% in 0.0333 secs.

### Test 01

#### KrakenD

    [vagrant@consumer ~]$ /usr/local/bin/test01
    100000 / 100000 Booooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00 %

    Summary:
      Total:  28.7424 secs.
      Slowest:  0.2781 secs.
      Fastest:  0.0009 secs.
      Average:  0.0287 secs.
      Requests/sec: 3479.1863
      Total Data Received:  10900000 bytes.
      Response Size per Request:  109 bytes.

    Status code distribution:
      [200] 100000 responses

    Response time histogram:
      0.001 [1]     |
      0.029 [71546] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.056 [26536] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.084 [1061]  |
      0.112 [392]   |
      0.140 [248]   |
      0.167 [93]    |
      0.195 [51]    |
      0.223 [12]    |
      0.250 [27]    |
      0.278 [33]    |

    Latency distribution:
      10% in 0.0213 secs.
      25% in 0.0231 secs.
      50% in 0.0245 secs.
      75% in 0.0295 secs.
      90% in 0.0432 secs.
      95% in 0.0469 secs.
      99% in 0.0771 secs.

#### Vulcand

    [vagrant@consumer ~]$ /usr/local/bin/test01
    100000 / 100000 Booooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00 %

    Summary:
      Total:  50.5294 secs.
      Slowest:  0.3426 secs.
      Fastest:  0.0010 secs.
      Average:  0.0505 secs.
      Requests/sec: 1979.0451
      Total Data Received:  10800000 bytes.
      Response Size per Request:  108 bytes.

    Status code distribution:
      [200] 100000 responses

    Response time histogram:
      0.001 [1] |
      0.035 [21120] |∎∎∎∎∎∎∎∎∎∎∎
      0.069 [71365] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.103 [5946]  |∎∎∎
      0.138 [168] |
      0.172 [74]  |
      0.206 [329] |
      0.240 [496] |
      0.274 [388] |
      0.308 [88]  |
      0.343 [25]  |

    Latency distribution:
      10% in 0.0290 secs.
      25% in 0.0378 secs.
      50% in 0.0490 secs.
      75% in 0.0571 secs.
      90% in 0.0665 secs.
      95% in 0.0733 secs.
      99% in 0.2057 secs.

#### Kong

    [vagrant@consumer ~]$ /usr/local/bin/test01
    100000 / 100000 Booooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00 %

    Summary:
      Total:  57.0194 secs.
      Slowest:  1.5978 secs.
      Fastest:  0.0076 secs.
      Average:  0.0569 secs.
      Requests/sec: 1753.7883
      Total Data Received:  13600000 bytes.
      Response Size per Request:  136 bytes.

    Status code distribution:
      [200] 100000 responses

    Response time histogram:
      0.008 [1]     |
      0.167 [97506] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.326 [2290]  |
      0.485 [103]   |
      0.644 [0]     |
      0.803 [0]     |
      0.962 [0]     |
      1.121 [0]     |
      1.280 [0]     |
      1.439 [17]    |
      1.598 [83]    |

    Latency distribution:
      10% in 0.0407 secs.
      25% in 0.0435 secs.
      50% in 0.0461 secs.
      75% in 0.0497 secs.
      90% in 0.0816 secs.
      95% in 0.1076 secs.
      99% in 0.2355 secs.

#### Tyk

    [vagrant@consumer ~]$ /usr/local/bin/test01
    100000 / 100000 Booooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo! 100.00 %

    Summary:
      Total:  221.5803 secs.
      Slowest:  5.6482 secs.
      Fastest:  0.0012 secs.
      Average:  0.2215 secs.
      Requests/sec: 451.3037
      Total Data Received:  10800000 bytes.
      Response Size per Request:  108 bytes.

    Status code distribution:
      [200] 100000 responses

    Response time histogram:
      0.001 [1]     |
      0.566 [90838] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      1.131 [4890]  |∎∎
      1.695 [2208]  |
      2.260 [1380]  |
      2.825 [383]   |
      3.389 [161]   |
      3.954 [86]    |
      4.519 [40]    |
      5.084 [6]     |
      5.648 [7]     |

    Latency distribution:
      10% in 0.0355 secs.
      25% in 0.0521 secs.
      50% in 0.0823 secs.
      75% in 0.1785 secs.
      90% in 0.5263 secs.
      95% in 0.9231 secs.
      99% in 2.1054 secs.

## Summary

### Requests per second

{{< gist kpacha 91caba50e47160f656069373b0f0605d "api-gateway-benchmark_test01_rps.csv">}}

### Response time distribution

*Time in milliseconds*

{{< gist kpacha 91caba50e47160f656069373b0f0605d "api-gateway-benchmark_test01_resp_time.csv">}}
