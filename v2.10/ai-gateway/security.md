---
lastmod: 2025-05-21
old_version: true
date: 2025-05-21
linktitle: AI Gateway Security
title: AI Security
description: Protect sensitive data and secure LLM access using KrakenD’s API-level AI security capabilities like isolated auth and data masking.
menu:
  community_v2.10:
    parent: "100 AI Gateway"
weight: 100

---
Protecting sensitive AI data and controlling access is essential for trustworthy AI workloads. KrakenD's AI Gateway integrates multiple layers of security at the edge to enforce zero-trust AI operations. From isolated authorization flows to data masking and exfiltration prevention, you can safeguard data traveling between clients and LLM providers without altering your existing API infrastructure.

The following key features are explained in this document:

- Isolated Authentication (JWT + API key separation)
- API Key Injection (backend-only exposure)
- Data Masking (request & response layers)
- Exfiltration Prevention Patterns

## Isolated Authentication
KrakenD separates the authentication flows between consumers and LLM on your AI endpoints. This way, you can implement different auth mechanisms and isolate consumers from vendor credentials. For instance:

![ai-gateway-isolated-auth.mmd diagram](/images/documentation/diagrams/ai-gateway-isolated-auth.mmd.svg)

Although **most LLM models use API Key authentication**, you don't need to adopt a less secure pattern when it comes to consumers, neither share your provider API key with them. Instead, you may want to integrate your user base through your existing Identity Provider and demand JWT tokens to use AI through the gateway. Then, you leave the API Key usage only between the gateway and the LLM without being exposed anywhere else.

KrakenD allows you to separate the two sides with the configuration that best suits your needs. The example above would use [JWT Validation](/docs/v2.10/authorization/jwt-validation/) at the endpoint level (what the consumer sees), and API Key protection at the backend level (what the gateway uses to talk to the LLM).

See:

- [JWT Validation](/docs/v2.10/authorization/jwt-validation/)
- API Key Protection (see below)

## API Key Protection
Even if your API is internal and not open to the world, you might want to safeguard API keys required by LLM providers and not share secrets with the rest of the company.

To do so, you can configure the gateway to **inject an Authorization header**; this API key remains in the configuration or an environment variable. This can be done using a [Martian modifier](/docs/v2.10/backends/martian/#header-modifier), as depicted below:

```json
{
  "version": 3,
  "$schema": "https://www.krakend.io/schema/krakend.json",
  "endpoints": [
    {
      "endpoint": "/chatgpt",
      "output_encoding": "no-op",
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
                "value": "Bearer YOUR_OPENAI_API_KEY"
              }
            }
          }
        }
      ]
    }
  ]
}
```
The example above would automatically add your OpenAI API key when calling the `/ChatGPT` endpoint on KrakenD. In this example, end-users would not be required to provide any kind of authentication (e.g., internal API), as the endpoint does not include JWT validation.

You can test the snippet above, replacing `YOUR_OPENAI_API_KEY` with your valid API KEY and test. You can also use enable [Flexible Configuration](/docs/v2.10/configuration/flexible-config/) and inject the API KEY from an env var using `"value": "Bearer {{  env "YOUR_OPENAI_API_KEY" }}"`.

With this config, you can call ChatGPT:

```
curl http://localhost:8080/chatgpt \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-3.5-turbo",
    "messages": [
      {"role": "system", "content": "Provide summarized answers and keep the language plain and simple"},
      {"role": "user", "content": "Explain how photosynthesis works."}
    ],
    "temperature": 0.7
  }'
```

If you have ChatGPT quota you should be able to see the response from ChatGPT without adding the authentication.

For more options on how to set static headers on external requests, see the [Martian API Key modifier](/docs/v2.10/backends/martian/#header-modifier)

## Data Masking & Data Loss Prevention
Thanks to the **[request and response transformation components](/docs/v2.10/request-response-manipulation/)** on KrakenD, you can easily apply data masking to protect sensitive information from being exposed in APIs by modifying the data before it leaves KrakenD. But also when you don't want end-users to pass sensitive content to the LLM.

There are two types of data masking:

- **Dynamic data masking** (e.g., enriched on an LLM)
- **Static data masking** (e.g., the gateway transforms the data itself based on patterns).

Data masking is especially useful when front-end clients, external consumers, or logs shouldn't see raw values like credit cards, SSNs, or personal identifiers.

### AI Static Response Masking
In KrakenD {{< badge >}}Enterprise{{< /badge >}}, **data masking in the response** can be done using the [regexp modifier](/docs/enterprise/endpoints/content-replacer/#regular-expression-modifier) inside the `modifier/response-body` namespace, while in the Open Source edition you must employ Lua Scripts. This allows you to define masking rules directly in your gateway configuration.

To accomplish this:

- You define a series of regex-based rules targeting the field containing the potentially sensitive content
- When the response passes through KrakenD, matching values are transformed
- The original sensitive values never leave the gateway.

Example {{< badge >}}Enterprise{{< /badge >}}:

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
      "endpoint": "/mask",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/__echo/123456789",
          "extra_config": {
            "modifier/response-body": {
              "modifiers": [
                {
                  "regexp": {
                    "@comment": "Example of masking SSN or credit numbers in the response body",
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
This is a simple example with a generic regular expression, but you can stack as many regexp patterns as needed by adding more modifiers to the array. The `field` attribute specifies which element of the response the replacement takes place in. As you can see, the `find` looks for any number of digits, hyphens, and spaces and replaces them with `XXX-` and the last four digits.

### AI Static Prompt Masking
You can also apply **static data masking in the request** before it reaches the backend or the LLM using the [Body generator](/docs/enterprise/backends/body-generator/) {{< badge >}}Enterprise{{< /badge >}}.

Suppose your users are sending a payload that could contain something sensitive, and you would like to apply a regexp before it passes to the LLM. For instance:

{{< terminal title="Term" >}}
curl -H'Content-Type: application/json' \
  -d '{"message": "123456789 is a number that should never be sent"}' \
  http://localhost:8080/req-mask
{{< /terminal >}}

Then, you can create the template of the payload as follows:


```gotmpl
{{- $message := index .req_body "message" -}}
{{- $message = regexReplaceAll "([\\d+\\-\\s]*)(\\d{4})" $message "XXX-${2}" -}}
{{/* Request JSON to the LLM: */}}
{
    "message": {{ $message | toJson }}
}
```

And include it in the configuration:

```json
{
  "endpoint": "/req-mask",
  "method": "POST",
  "backend": [
    {
      "url_pattern": "/__echo/test",
      "host": ["http://localhost:8080"],
      "extra_config": {
        "modifier/request-body-generator": {
          "path": "body_mask.tmpl",
          "content_type": "application/json",
          "debug": true
        }
      }
    }
  ]
}
```

The payload the LLM would receive under these conditions would be

```json
{
    "message": "XXX-6789 is a number that should never be sent."
}
```

### LLM-based Request and Response Transformer
A more sophisticated way of masking incoming or outgoing data is to delegate the request or the response to an LLM that will manipulate the request or the response using more sophisticated mechanisms.

To do this, you have to configure a [sequential proxy](/docs/v2.10/endpoints/sequential-proxy/) where you:

1. Add the LLM in the first place to modify a request
2. Add the LLM in the second place (after querying your backend) to modify the response.

## Exfiltration Prevention
Exfiltration prevention in KrakenD can be approached by implementing a combination of configuration rules and optional custom plugins that restrict, sanitize, or monitor outbound data to ensure that no unauthorized or sensitive data leaves your perimeter via the API gateway.

As with data masking, exfiltration prevention can be static or dynamic.

Here are some indications to avoid exfiltration using **static patterns**:

- Use an [allow list](/docs/v2.10/backends/data-manipulation/#allow) for APIs that should return a limited set of attributes
- Use [Security Policies](/docs/enterprise/security-policies/) {{< badge >}}Enterprise{{< /badge >}} to check necessary roles and conditions to deliver content
- Use [Regex Masking or Redaction](/docs/v2.10/ai-gateway/security/#data-masking--data-loss-prevention) on fields that must not be exfiltrated in full.
- Use custom logic in [request and response modifier plugins](/docs/v2.10/extending/plugin-modifiers/)
- Instead of allowing open LLM queries, use [prompt templates](/docs/enterprise/ai-gateway/governance/#prompt-templates) {{< badge >}}Enterprise{{< /badge >}}

For **dynamic operation**, you can delegate the output to an LLM that will refine the response. To do that, you can add a [sequential proxy](/docs/v2.10/endpoints/sequential-proxy/) step as the second backend, as explained in the chapter above.
