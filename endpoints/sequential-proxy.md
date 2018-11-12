---
lastmod: 2018-11-11
date: 2018-11-11
toc: true
linktitle: Sequential Proxy
title: Sequential Proxy
weight: 60
menu:
  documentation:
    parent: endpoints
---
The best experience consumers can have with KrakenD API is by letting the system fetch all the data from the different backends concurrently at the same time. However, there are times when you need to **delay a backend call** until you can inject as input the result of a previous call.

The sequential proxy allows you to **chain backend requests**.

# Chaining the requests
All you need to enable the sequential proxy is add in the endpoint definition the following configuration:

    "endpoint": "/hotels/{id}",
    "extra_config": {
          "github.com/devopsfaith/krakend/proxy": {
              "sequential": true
          }
      }

When the sequential proxy is enabled, the `url_pattern` of every backend can use a new variable that references the **resp**onse of a previous API call. The variable has the following construction:

    {resp0_XXXX}

Where `0` is the index of the specific backend you want to access ( `backend` array), and where `XXXX` is the attribute name you want to inject from the previous call.


## Example
It's easier to understand with an example:

The `/hotels/{hotel_id}` is a backend call that returns data for the requested hotel, including a `destination_id` that is a relationship identifier. The output is like the following:

    {
      "hotel_id": 25,
      "name": "Hotel California",
      "destination_id": 1034
    }

Another backend call, `/destinations/{destination_id}`, returns all the possible destinations for a given numeric ID:

    {
      "destination_id": 1034,
      "destinations": [
        { ... }, { ... }
      ]
    }

We want the API Gateway to expose `/hotels/{id}` and return in the same call both the hotel data and its relationship with the destinations. The code looks like this:

    "endpoint": "/hotels/{id}",
    "backend": [
        { <--- Index 0
            "host": [
                "https://hotels.api"
            ],
            "url_pattern": "/hotels/{id}"
        },
        { <--- Index 1
            "host": [
                "https://hotels.api"
            ],
            "url_pattern": "/destinations/{resp0_destination_id}"
        }
    ],
    "extra_config": {
        "github.com/devopsfaith/krakend/proxy": {
            "sequential": true
        }
    }

The key here is the variable `{resp0_destination_id}` that references the `destination_id` to the response of the backend number `0` (first).
