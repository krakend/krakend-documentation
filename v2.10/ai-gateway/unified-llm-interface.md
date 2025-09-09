---
lastmod: 2025-05-21
old_version: true
date: 2025-05-21
linktitle: Unified LLM Interface
title: Unified LLM Interface
description: Simplify multi-LLM integration by exposing a single, consistent API through KrakenD’s unified LLM interface.
menu:
  community_v2.10:
    parent: "100 AI Gateway"
weight: 130

---
The Unified LLM Interface of KrakenD allows you to interact with one or more LLMs, removing the complexity of dealing with LLMs for the end user. It allows you to set the grounds to communicate with LLMs, authenticate, and treat requests and responses back and forth.

## LLM Routing
KrakenD’s AI Proxy and LLM routing feature enables you to distribute AI requests across multiple Large Language Model providers or instances. This ensures high availability, optimized performance, cost efficiency, developer abstraction, and compliance.

LLM Routing on KrakenD supports both single-provider and multi-provider. You can configure endpoints that can connect to one LLM model, or you can have endpoints that, based on policies, choose the right model, or they even make simultaneous requests to different providers to aggregate their responses.

See Multi-LLM routing below (where single-provider routing is a one-option version of the multi-LLM).

## Multi-LLM Routing
To implement multi-LLM routing in KrakenD, like dynamically selecting between different Large Language Model (LLM) providers like OpenAI, Mistral, Claude, or custom models, there are several clean, scalable strategies depending on how you want to choose the provider:

- Header-based routing
- JWT claim-based routing
- Path-based routing
- Other non-standard routing logic



### Header-based LLM routing
This would be a frequent choice where end-users set a specific header with the desired model to use, and from here, the gateway chooses the route. For instance, say you want to integrate with two providers and let the consumer of the API Gateway decide which one to use based on a header:

![ai-gateway-proxy-header.mmd diagram](/images/documentation/diagrams/ai-gateway-proxy-header.mmd.svg)

The consumer would send a request like this:

```
POST /multi-llm
X-LLM: openai
```

And the gateway uses the header's value to route to the appropriate backend.

To implement this **there are several strategies you can use**. Open Source users can go with [Conditional requests](/docs/v2.10/endpoints/common-expression-language-cel/) or [Lua Scripts](/docs/v2.10/endpoints/lua/). Enterprise users can also use the [Dynamic routing](/docs/enterprise/endpoints/dynamic-routing/). In the following example configuration, an entry-point `/multi-llm` receives the user request and evaluates the header. Then it loads the content internally from another endpoint `/_llm/openai` or `/_llm/gemini`, routing to the proper vendor, Google Gemini or OpenAPI. In addition, the request modifier component allows you to set the payload as each vendor expects it:

```json
{
  "version": 3,
  "$schema": "https://www.krakend.io/schema/krakend.json",
  "host": [
    "localhost:8080"
  ],
  "echo_endpoint": true,
  "endpoints": [
    {
      "endpoint": "/multi-llm",
      "input_headers": [
        "X-Llm"
      ],
      "extra_config": {
        "security/policies": {
          "auto_join_policies": true,
          "debug": false,
          "req": {
            "@comment": "Validates that the X-LLM header belongs to a known LLM",
            "policies": [
              "req_headers['X-Llm'][0] in ['gemini','openai']"
            ]
          }
        }
      },
      "backend": [
        {
          "url_pattern": "/_llm/{input_headers.X-Llm}"
        }
      ]
    },
    {
      "endpoint": "/_llm/gemini",
      "backend": [
        {
          "host": [
            "https://generativelanguage.googleapis.com"
          ],
          "url_pattern": "/v1beta/models/gemini-pro:generateContent?key=YOUR_API_KEY",
          "extra_config": {
            "modifier/body-generator": {
              "path": "_llms/gemini.tmpl",
              "content_type": "application/json",
              "debug": true
            }
          }
        }
      ]
    },
    {
      "endpoint": "/_llm/openai",
      "method": "POST",
      "backend": [
        {
          "host": [
            "https://api.openai.com"
          ],
          "url_pattern": "/v1/chat/completions",
          "extra_config": {
            "modifier/martian": {
              "header.Modifier": {
                "scope": [
                  "request"
                ],
                "name": "Authorization",
                "value": "Bearer YOUR_OPENI_API_KEY"
              }
            }
          }
        }
      ]
    },
    {
      "endpoint": "/resp-mask",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/__echo/123456789",
          "extra_config": {
            "modifier/response-body": {
              "modifiers": [
                {
                  "regexp": {
                    "@comment": "Credit card masking example for consecutive digits",
                    "field": "req_uri_details.path",
                    "find": "\\b(\\d{13,16})\\b",
                    "replace": "XXX-CREDIT-CARD-XXX"
                  }
                },
                {
                  "regexp": {
                    "@comment": "Mask the SSN numbers in the response body",
                    "field": "req_uri_details.path",
                    "find": "([\\d+\\-\\s]*)(\\d{4})",
                    "replace": "XXX-$2"
                  }
                }
              ]
            }
          }
        }
      ]
    }
  ]
}
```
Notice that the request body generator templates are not included in this example; find them below. Open-source users can replace body generators with [Lua scripts](/docs/v2.10/endpoints/lua/).

### JWT claim-based LLM routing
Similarly to header routing, you can use data in claims to route to the proper LLM. For instance, when you issue a JWT token, you could add a claim that specifies what LLM to use based on role, business policies, etc.

As opposed to header-based routing, this method is transparent to the user, who does not have any control over the model used, because it is enforced by a policy in the Identity Provider.

It would work with the same configuration you use in the header-based routing, only that you use the JWT instead:

```json
{
  "url_pattern": "/_llm/{jwt.preferred_ai_model}"
}
```

### Path-based LLM routing
This is the simplest way of routing because you directly declare the endpoints you want to use, e.g.:

- `/llm/openai`
- `/llm/mistral`

Then the user calls one endpoint or the other, and the API offers one endpoint per provider. You will need at least to configure [Isolated Authentication](/docs/v2.10/ai-gateway/security/#isolated-authentication) or [API Key protection](/docs/v2.10/ai-gateway/security/#api-key-protection).

## Vendor Transformation
As briefly introduced in the Header-based routing, each vendor will require a different payload and communication style. On KrakenD you can implement completely different models while you keep the user away from this process.

The way to adapt the payload sent to each LLM, you need to pass the right transformation template to each LLM. This is achieved with the [body generator](/docs/enterprise/backends/body-generator/) takes care of this job.

Examples of vendor templates, for a user payload containing a `{"content": ""}` would be:

### Google Gemini transformation template
```gotmpl
{
    "contents": [
      {
        "parts": [
          { "text": {{ .req_body.content | toJson }} }
        ]
      }
    ]
  }
```
### ChatGPT (OpenAI) transformation template
```gotmpl
{
    "model": "gpt-4o",
    "messages": [
      { "role": "user", "content": {{ .req_body.content | toJson }} }
    ]
  }
```

### Mistral transformation template
E.g. via Mistral-hosted or OpenRouter:

```gotmpl
{
    "model": "mistral-medium",
    "messages": [
      { "role": "user", "content": {{ .req_body.content | toJson }} }
    ]
  }
```

### Deepseek transformation template
```gotmpl
{
    "model": "deepseek-chat",
    "messages": [
      { "role": "user", "content": {{ .req_body.content | toJson }} }
    ]
  }
```

### LLaMa transformation template
```gotmpl
{
    "model": "meta-llama-3-70b-instruct",
    "messages": [
      {
        "role": "user",
        "content": {{ .req_body.content | toJson }}
      }
    ]
  }
```
Or `"model": "llama3"` for **Ollama**


### Anthropic transformation template
```gotmpl
{
  "model": "claude-3-sonnet-20240229",
  "max_tokens": 1024,
  "temperature": 0.7,
  "messages": [
    {
      "role": "user",
      "content": {{ .req_body.content | toJson }}
    }
  ]
}
```

### Cohere transformation template
```gotmpl
{
  "model": "command-r-plus",
  "chat_history": [],
  "message": {{ .req_body.content | toJson }},
  "temperature": 0.5
}
```

### Other providers
The templates above are transformation examples that you can extend to add prompts, introduce placeholders, or support non-listed LLM providers.
