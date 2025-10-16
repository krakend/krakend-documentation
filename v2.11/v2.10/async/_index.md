---
lastmod: 2022-01-21
old_version: true
old_version: true
date: 2022-01-21
linktitle: Event-Driven Async Agents
title: "Event-Driven API Gateway: Async Agents"
description: Learn how to leverage asynchronous processing in KrakenD API Gateway to implement an event based pattern
weight: 500
dark_header_image: true
images:
  - /images/documentation/async-agents.png
menu:
  community_v2.10:
    parent: "050 Non-REST Connectivity"
meta:
  since: v2.0
  source: https://github.com/luraproject/lura
  scope:
  - service
  log_prefix:
  - "[ASYNC: AgentName][AgentType]"
---
**Async agents** are routines listening to queues or PubSub systems that react to new events and push data to your backends. Through async agents, you can start a lot of consumers to process your events autonomously.

As opposed to endpoints, async agents do not require users to request something to trigger an action. Instead, the agents connect to an event messaging system and fire an action when an event is delivered.

**An async agent can do everything an endpoint can do**. You can use plugins, apply transformations and manipulations, scripting, stub data, parallel or sequential calls to multiple backends, jsonschema, OAuth2 client credentials, rate limiting, circuit breaking, validations, lambda, and a long long etcetera.

The obvious limitation is that you cannot use HTTP request functionality (e.g: CORS or JWT validation) as you don't have any user doing an HTTP request, but an automatic trigger from KrakenD when an event pops in.

## When do you need Async Agents
You are trying to implement an event based pattern, such as:

- The Saga Pattern
- Event sourcing

## How Async agents work
When KrakenD starts, it reads the `async_agent` list in the configuration and creates the declared agents. An agent is an application thread that can use one or multiple workers connecting to a queue or PubSub system (consumers). KrakenD contacts the defined backend(s) list passing the event data when a new message kicks in. You might decide to add manipulations, validations, filtering, or any other backend functionality supported by KrakenD.

The backend(s) receive the event from the agent as part of the body. Depending on the driver and configuration, when a backend fails to process the request, you can tell KrakenD to reinject the message (`Nack`) to retry the message later by any other worker. Notice that when working with Nack, if KrakenD is the only consumer and your backend fails to process the message continuously, KrakenD will reinsert the message into the queue over and over, and could lead to an infinite loop of messages if no consumer empties these messages.

Notice that as it happens with the endpoints, the messages you consume can be sent in parallel or sequentially to multiple services.


## Configuration

The `async_agent` entry is **an array** with all the different agents you want to have running. Each configuration needs to declare in the `extra_config` the connection driver you want to use. like this:

```json
{
    "version": 3,
    "async_agent": [
        {
            "name": "cool-agent",
            "connection": {
                "max_retries": 10,
                "backoff_strategy":"exponential-jitter",
                "health_interval": "10s"
            },
            "consumer": {
                "topic": "*",
                "workers": 1,
                "timeout": "150ms",
                "max_rate": 0.5
            },
            "backend": [
                {
                    "host": [
                        "http://127.0.0.1:8080"
                    ],
                    "url_pattern": "/__debug/"
                }
            ],
            "extra_config": {
                "async/amqp": {
                    "host": "amqp://guest:guest@localhost:5672/",
                    "name": "krakend",
                    "exchange": "foo",
                    "durable": true,
                    "delete": false,
                    "exclusive": false,
                    "no_wait": true,
                    "prefetch_count": 5,
                    "auto_ack": false,
                    "no_local": true
                }
            }
        }
    ]
}
```
The configuration accepts the following parameters:

{{< schema version="v2.10" data="async_agent.json" property="items" title="Properties of an Async Agent object" >}}

When agents are defined, their activity is shown in the health endpoint with the name of the agent you have chosen.  The health endpoint will show for each agent, when was the last time the agent reported itself as alive. The frequency of this checking is as defined in the `health_interval`.

Check [how agents report in the health endpoint](/docs/v2.11/v2.10/service-settings/health/)

### Backoff strategies
The `backoff_strategies` you can set are defined below:

- `linear`: The delay time (`d`) grows linearly after each failed retry (`r`) using the formula `d = r`. E.g.: 1st failure retries in 1s, 2nd failure in 2s, 3rd in 3s, and so on.
- `linear-jitter`: Similar to `linear` but adds or subtracts a random number: `d = r ± random`. The randomness prevents all agents connected to a mutual service from retrying simultaneously as all have a slightly different delay. The random number never exceeds `±r*0.33`
- `exponential`: Multiplicatively increase the time between retries using `d = 2^r`. E.g: `2s`, `4s`, `8s`, `16s`...
- `exponential-jitter`: Same as exponential, but adds or subtracts a random number up to 33% of the value using `d = 2^r ± random`. This is the **preferred strategy** when you want to protect the system you are consuming.
- **Fallback**: When the strategy is missing or none of the above (e.g.:`fallback`) then it will use **constant** backoff strategy `d=1`. Will retry after one second every time.
