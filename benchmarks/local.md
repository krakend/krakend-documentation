---
lastmod: 2016-10-28
date: 2016-10-28
linktitle: Local machine
title: Local Benchmarks
description: Performance tests of KrakenD
weight: 15
menu:
  community_current:
    parent: "160 Benchmarks"
---

## Hardware

    Model MacBook Pro (MacBookPro11,4) - August 2015
    Processor:    Intel Core i7 2,2 GHz

## Setup

For this test, we stored this configuration at `krakend.json`

    {
      "version": 1,
      "endpoints": [
        {
          "endpoint": "/foo",
          "method": "GET",
          "backend": [
            {
              "url_pattern": "/__debug/bar",
              "host": [
                "http://127.0.0.1:8080"
              ]
            }
          ],
          "concurrent_calls": "1",
          "max_rate": 100000
        }
      ],
      "oauth": {
        "disable": true
      },
      "cache_ttl": "5m",
      "timeout": "5s"
    }

And we started the KrakenD with this cmd:

    $ ./krakend run --config krakend.json -d > /dev/null

Response from the 'debug backend':

    $ curl -i http://127.0.0.1:8080/__debug/bar
    HTTP/1.1 200 OK
    Content-Type: application/json; charset=utf-8
    Date: Mon, 28 Nov 2016 15:42:16 GMT
    Content-Length: 19

    {"message":"pong"}

Response from the KrakenD:

    $ curl -i http://127.0.0.1:8080/foo
    HTTP/1.1 200 OK
    Cache-Control: public, max-age=300
    Content-Type: application/json; charset=utf-8
    X-Krakend: Version 0.3.8
    Date: Mon, 28 Nov 2016 15:42:20 GMT
    Content-Length: 19

    {"message":"pong"}
   
## Results

    $ hey -c 200 -n 100000 http://127.0.0.1:8080/foo
    8217 requests done.
    17379 requests done.
    25928 requests done.
    34664 requests done.
    43290 requests done.
    52069 requests done.
    61057 requests done.
    69954 requests done.
    78815 requests done.
    87565 requests done.
    96615 requests done.
    All requests done.

    Summary:
      Total:  5.6980 secs
      Slowest:  0.0494 secs
      Fastest:  0.0003 secs
      Average:  0.0113 secs
      Requests/sec: 17549.8782
      Total data: 1900000 bytes
      Size/request: 19 bytes

    Status code distribution:
      [200] 100000 responses

    Response time histogram:
      0.000 [1]     |
      0.005 [974]   |∎
      0.010 [42346] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.015 [44698] |∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎∎
      0.020 [10393] |∎∎∎∎∎∎∎∎∎
      0.025 [1242]  |∎
      0.030 [180]   |
      0.035 [121]   |
      0.040 [22]    |
      0.045 [15]    |
      0.049 [8]     |

    Latency distribution:
      10% in 0.0082 secs
      25% in 0.0091 secs
      50% in 0.0106 secs
      75% in 0.0130 secs
      90% in 0.0155 secs
      95% in 0.0173 secs
      99% in 0.0212 secs

## Summary

{{< gist kpacha 91caba50e47160f656069373b0f0605d "local_100_500_concurrency.csv">}}