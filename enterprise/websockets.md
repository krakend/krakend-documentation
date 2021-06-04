---
lastmod: 2020-05-25
date: 2020-05-25
linktitle: WebSockets
title: Handling WebSockets connections
weight: 140
enterprise: true
menu:
  community_current:
    parent: "040 Endpoint Configuration "
images:
- /images/documentation/krakend-websockets.png
---

KrakenD Enterprise supports communications using the WebSocket Protocol (RFC-6455) to enable two-way communication between a client to a backend host through the API gateway. This technology aims to provide a mechanism for browser-based applications that need two-way communication with servers that do not rely on opening multiple HTTP connections.

KrakenD has the capability of **multiplexing**. Each individual end-client (e.g., Desktop, Mobile device) establishes a connection with the gateway directly, and KrakenD opens a single channel with the backend host to handle all its connected clients.

The backend decides whether to send the responses to a specific set of client(s) or to broadcast to everyone. This decision depends on the **message format** 

## Broadcast vs. directed communication.

No matter how many clients connect to KrakenD through WebSockets, that KrakenD keeps **one connection with the backend server** (as depicted above). For instance, you might have 1000 concurrent users in a chat room, with 1000 sockets opened against KrakenD, but KrakenD still communicates with your backend using one channel.

Depending on your business type, you might have two different needs to communicate with the users connected to the gateway: 

- Send a message to all your clients (**broadcast**)
- Send a message only to some user(s) (**directed communication**)

By default, unrecognized messages received by KrakenD do broadcast to all clients. The **message format** allows you to specify how you would like KrakenD to process your backend responses (broadcast to all connected users, or just to a specific session).

## Message format
The message format is the mechanism used by KrakenD to determine if a message needs broadcasting or addressing a specific set of clients.

The recognized message format for the bidirectional communication is a JSON object with the following fields:

- `body`: Mandatory. A **base64** representation of the content.
- `session`: The session is a filter to determine who is the sender or the receiver of a message.
- `url`: The affected KrakenD endpoint.

### KrakenD message for the backend
When the client interacts with KrakenD, the gateway sends messages to the backend with a structure like the following:

    { 
        "url": "/chat/krakend",
        "session": { 
            "uuid": "0b251b07-5611-49e5-b69f-cf2cb8d339d6",
            "Room":"krakend",
        },
        "body": "SGVsbG8gV29ybGQh"
    }

The client sent what you can see in the `body` (*"Hello World!"* base64 encoded).

The rest of the fields are contextual metadata for your convenience, so the backend can determine who is doing the call and from which originating endpoint.

Essential observations on the example above are:

- The `body` is **base64** encoded.
- The `session` contains information about the client making the request. At least you always receive an `uuid` which is a randomly assigned by KrakenD to the client when a new session starts.
- If your endpoint contains placeholders (e.g., as in `/chat/{room}`), the placeholder parameters are available under `session`, but using the first letter uppercased. In this example, `Room` holds the value of `{room}`).


### Backend message for KrakenD
The return from your WS backend needs to use the expected format if you don't want to broadcast it.

The response `body` is mandatory, and KrakenD forwards to all connected clients matching the filters you pass. If you don't pass any filters, then we are talking about broadcasting. The filters are a combination of the `session` and the `url`.

If for instance, you only want to communicate with a specific user, you would have an answer like this one:

    {
        "session": { "uuid": "0b251b07-5611-49e5-b69f-cf2cb8d339d6"},
        "body": "SGVsbG8gV29ybGQh"
    }

If you want to communicate with all users connected to an endpoint, then you could use:

    {
        "url": "/chat/krakend",
        "body": "SGVsbG8gV29ybGQh"
    }

And a broadcast:

    {
        "body": "SGVsbG8gV29ybGQh"
    }

Notice that when the JSON fields `url` or `session` exist, the `body` is sent to the specific subgroup instead of being broadcasted.

## Configuration
The configuration to enable WebSockets is straightforward; the only requirement is to include the `github.com/devopsfaith/krakend-websocket` namespace at the `endpoint` level, and that you declare one backend host (and only one) using the `ws://` or `wss://` schema.

The flag `"disable_host_sanitize": true` is also necessary for the backend.

Here there is an example:

    {
      "endpoint": "/chat/{room}",
      "backend": [
        {
          "url_pattern": "/ws",
          "disable_host_sanitize": true,
          "host": [ "ws://localhost:8081" ]
        }
      ],
      "extra_config":{
        "github.com/devopsfaith/krakend-websocket": true
      }
    }

## Handshaking
The WS protocol consists of an opening handshake followed by basic message framing, layered over TCP. 

The handshake process is straightforward but necessary for KrakenD to determine if the backend server is alive. It's managed by KrakenD itself, and it consists in nothing else than opening a WebSocket against the backend server and trying to find a response (like a ping/pong). We can simplify it as: 

    krakend => {"msg":"KrakenD WS proxy starting"}
    backend => OK

After the successful handshake, KrakenD is ready to start serving and communicating with WS.

### Connectivity issues
The nature of WebSocket connections is that they have kind of a "state" and use a lasting connection. There are implications to be aware of when connectivity issues or downtimes arise.

**Issues during startup**:
The most visible problem of all. If, for whatever reason, the WebSocket on the backend server is not available **during KrakenD startup**, KrakenD won't start. This event shows in the console with a *CRITICAL* message like this one:

    [KRAKEND] 2018/12/26 - 12:31:24.708 ▶ CRITIC websocket.Dial ws://localhost:8081/ws: dial tcp [::1]:8081: connect: connection refused

If KrakenD relies on WebSocket communication, make sure that the backends are alive when KrakenD starts, or otherwise, the gateway won't.

**During regular operation**:
If, on the other hand, the handshake succeeded, but at a given point in time, the backend server with the WS dies, the affected endpoint becomes non-operational.

All clients connected to KrakenD during the downtime of your backend's WebSocket keep their connection with KrakenD, despite the fact that KrakenD is unable to pass any data from/to the backend server. The error indicating this behavior is:

    [KRAKEND] 2018/12/26 - 15:34:07.178 ▶ WARNIN WS client: EOF

Even after the backend service has resumed its service, the affected endpoint(s) won't be fully available. The connection remains unstable until the KrakenD service is restarted again with a successful connection. So in the case of backend downtime, the recommended course of action is to restart or redeploy KrakenD to minimize issues.