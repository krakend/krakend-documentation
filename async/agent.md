---
lastmod: 2021-06-13
date: 2019-09-15
linktitle:  Async Agents
title: Async agents
weight: 90
images:
  - /images/documentation/async-agents.png
menu:
  community_current:
    parent: "060 Async Agents"
meta:
  since: 2.0
  source: https://github.com/luraproject/lura
  scope:
  - service
  log_prefix:
  - "[ASYNC: AgentName][AgentType]"
---
**Async agents** are routines listening to queues or PubSub systems that react to new events and push data to your backends. Through async agents, you can start a lot of consumers to process your events autonomously.

As opposed to endpoints, async agents do not require users to request something to trigger an action. Instead, the agents connect to an event messaging system and fire an action when a new event enters the queue by themselves.

**An async agent can do everything an endpoint can do**. You can use plugins, apply transformations and manipulations, scripting, stub data, parallel or sequential calls to multiple backends, jsonschema, OAuth2 client credentials, rate limiting, circuit breaking, validations, lambda, and a long long etcetera.

The obvious limitation is that you cannot use HTTP request functionality (e.g: CORS or JWT validation) as you don't have any user doing an HTTP request, but an automatic trigger from KrakenD when an event pops in.

## How Async agents work
When KrakenD starts, it reads the `async_agent` list in the configuration and creates the declared agents. An agent is a standalone process with one or multiple workers that connect to a queue or PubSub system (consumers). KrakenD contacts the defined backend(s) list passing the event data when a new message kicks in. You might decide to add manipulations, validations, filtering, or any other backend functionality supported by KrakenD.
The backend receives the event from the agent as part of the body. If the backend fails to process the request, KrakenD reinjects the message (`Nack`) to retry the message later by any other worker.

{{< note title="Infinite loops" type="warning" >}}
If KrakenD is the only consumer and your backend fails to process the message, KrakenD will reinsert the message into the queue. This scenario can lead to an infinite loop of messages if no consumer empties the message.
{{< /note >}}

## Configuration

The configuration needs to declare in the `extra_config` the connection driver you want to use. like this:

{{< highlight json >}}
{
    "version": 3,
    "async_agent": [
        {
            "name": "cool-agent",
            "topic": "*",
            "workers": 1,
            "max_retries": 10,
            "backoff_strategy":"exponential-jitter",
            "timeout": "150ms",
            "health_interval": "10s",
            "backend": [
                {
                    "host": [
                        "http://127.0.0.1:8080"
                    ],
                    "url_pattern": "/__debug"
                }
            ],
            "extra_config": {
                "github.com/devopsfaith/krakend-amqp/consume": {
                    "name": "krakend",
                    "host": "amqp://guest:guest@localhost:5672/",
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
{{< /highlight >}}
The configuration accepts the following parameters:
- `name` (*string*): A unique name for this agent. KrakenD shows it in the [health endpoint](/docs/service-settings/health/) and logs and metrics. KrakenD does not check collision names, so make sure each agent has a different name.
- `topic` (*string*): The topic name you want to consume. The syntax depends on the driver. Examples for AMQP: `*`, `mytopic`, `lazy.#`, `*`, `foo.*`
- `workers` (*integer*): The number of workers (consuming processes) you want to start simultaneously for this agent. Defaults to `1`.
- `max_retries` (*integer*): The number of times you will allow KrakenD to retry a broken connection. `0` for unlimited retries.
- `backoff_strategy` (*string*): When the connection to your event source gets interrupted for whatever reason, KrakenD keeps trying to reconnect until it succeeds or until it reaches the `max_retries`. The backoff strategy defines the delay in seconds in between consecutive failed retries:
    - `linear`: The delay time (`d`) grows linearly after each failed retry (`r`) using the formula `d = r`. E.g.: 1st failure retries in 1s, 2nd failure in 2s, 3rd in 3s, and so on.
    - `linear-jitter`: Similar to `linear` but adds or subtracts a random number: `d = r ± random`. The randomness prevents all agents connected to a mutual service from retrying simultaneously as all have a slightly different delay. The random number never exceeds `±r*0.33`
    - `exponential`: Multiplicatively increase the time between retries using `d = 2^r`. E.g: `2s`, `4s`, `8s`, `16s`...
    - `exponential-jitter`: Same as exponential, but adds or subtracts a random number up to 33% of the value using `d = 2^r ± random`. This is the **preferred strategy** when you want to protect the system you are consuming.
    - **Fallback**: When the strategy is missing or none of the above (e.g.:`fallback`) then it will use **constant** backoff strategy `d=1`. Will retry after one second every time.
- `timeout` (*duration*): The maximum time the agent will wait to process an event sent to the backend. If the backend fails to process it, the message is reinserted for later consumption.
- `health_interval`": The time between pings checking that the agent is connected to the queue and alive. Defaults to `1s`. Regardless of the health interval, if an agent fails, KrakenD will restart it again immediately.
- `backend` (*list*): The [backend definition](/docs/backends/overview/) (as you might have in any endpoint) indicating where the event data is sent. It is a full backend object definition, with all its possible options, transformations, filters, validations, etc.
- `extra_config` (*map*): In the extra configuration you have to define the driver that connects to your queue or PubSub system.
