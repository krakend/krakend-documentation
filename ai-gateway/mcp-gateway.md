---
lastmod: 2025-10-29
date: 2025-10-29
linktitle: MCP Gateway
title: MCP Gateway
description: Protect a third-party MCP server through KrakenD and add security, observability, and traffic management features.
menu:
  community_current:
    parent: "100 AI Gateway"
weight: 300
notoc: true
images:
- /images/documentation/diagrams/ai-gateway-mcp-gateway.mmd.svg

---
The Model Context Protocol (MCP) Gateway in KrakenD enables transparent communication between an **MCP agent** and a **third-party MCP server** through a `no-op` endpoint. This setup allows KrakenD to forward messages with minimal processing while enriching the communication path with additional layers and functionality such as:

- Adding [authorization](/docs/authorization/) and other security options
- Add [observability](/docs/telemetry/)
- [Throttle](/docs/throttling/) the usage of the MCP
- Add [CORS](/docs/service-settings/cors/)
- Add [IP filtering](/docs/enterprise/throttling/ipfilter/) {{< badge >}}Enterprise{{< /badge >}}
- Add [security policies](/docs/enterprise/security-policies/) or [CEL expressions](/docs/endpoints/common-expression-language-cel/) with business logic
- Move the passing of headers to the gateway and make the client unaware with [Martian](/docs/backends/martian/)
- Limit the [payload size](/docs/enterprise/endpoints/maximum-request-size) of the requests {{< badge >}}Enterprise{{< /badge >}}

- And a long etcetera, check the documentation menu.

Use KrakenD's MCP Gateway to **connect MCP agents to external MCP servers** that lack sufficient Enterprise-ready features, leveraging KrakenD as a secure, observable, and manageable gateway. This feature relays MCP agent content directly to the MCP server while enabling configurable middlewares to enhance protocol flow control and monitoring.

KrakenD can also be the [MCP Server](/docs/enterprise/ai-gateway/mcp-server/) if needed {{< badge >}}Enterprise{{< /badge >}}

Example use cases:

- Secure MCP Agent Communication: Inject authentication and authorization to restrict MCP agent interactions with third-party MCP servers.
- Rate Limiting MCP Traffic: Protect MCP servers from overload by applying rate limits at the gateway.
- Observability and Logging: Add tracing and metrics on MCP message exchanges to improve operational insight.
- Enterprise MCP Server: For deployments needing an in-house MCP server, KrakenD Enterprise includes the MCP Server feature that can be integrated or proxied using this gateway.

## MCP Gateway configuration
To configure an endpoint in KrakenD acting as an MCP gateway to forward raw MCP messages, you only need a configuration like this:

```json
{
  "endpoints": [
    {
      "endpoint": "/mcp",
      "output_encoding": "no-op",
      "input_headers": [
        "HeaderNeededByYourMCP"
      ],
      "method": "POST",
      "backend": [
        {
          "url_pattern": "/mcp",
          "host": [
            "https://third-party-mcp-server.local"
          ]
        }
      ]
    },
    {
      "endpoint": "/mcp",
      "output_encoding": "no-op",
      "input_headers": [
        "HeaderNeededByYourMCP"
      ],
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/mcp",
          "host": [
            "https://third-party-mcp-server.local"
          ]
        }
      ]
    },
    {
      "endpoint": "/mcp",
      "output_encoding": "no-op",
      "input_headers": [
        "HeaderNeededByYourMCP"
      ],
      "method": "DELETE",
      "backend": [
        {
          "url_pattern": "/mcp",
          "host": [
            "https://third-party-mcp-server.local"
          ]
        }
      ]
    }
  ]
}
```
Line-by-Line:

- `method`: Three endpoints, as we want to allow MCP communication through `GET`, `POST` and `DELETE`. Being the `POST` the most important.
- `output_encoding`: The `no-op` encoding allows transparent MCP communication. This is a must.
- `input_headers`: Depending on the MCP technology, you might need forwarding headers to the MCP server. Or you can add [Martian](/docs/backends/martian/) and inject them at the gateway level if they have constant values and you don't want the MCP agent to know.
- `endpoint`: `/mcp` defines the KrakenD endpoint exposed to MCP agents, whatever route you want this to be.
- `host`: Remote MCP server address to forward MCP messages.

{{< note title="Supported MCP Server types" type="info" >}}
The MCP server connected to KrakenD must be HTTP-based, like JSONRPC or SSE. Stdio communication is out of scope for this functionality. If you plan to use SSE, make sure to add also a `timeout` with a value large enough to not close the session on the `GET` method only.
{{< /note >}}


From here, you can add Authorization, Telemetry, Rate Limiting, or any other KrakenD component that is compatible with no-op.

KrakenD's MCP Gateway feature enables infrastructure teams to transparently connect MCP agents to third-party MCP servers while adding vital control layers such as authorization, observability, and rate limiting. The no-op proxy behavior ensures the integrity of the MCP protocol, enabling integration and operational management.

For advanced MCP deployments, KrakenD Enterprise offers a native [MCP Server](/docs/enterprise/ai-gateway/mcp-server/) option that complements or replaces third-party MCP resources.