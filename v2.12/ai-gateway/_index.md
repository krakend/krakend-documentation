---
lastmod: 2025-05-21
old_version: true
date: 2025-05-21
linktitle: AI Gateway Overview
title: AI Gateway - API Gateway for LLMs
description: Discover how KrakenD transforms your API Gateway into a full-featured AI Gateway, with token enforcement, multi-LLM orchestration, and LLM security.

###
menu:
  community_v2.12:
    parent: "100 AI Gateway"
weight: -1
dark_header_image: true
images:
  - /images/documentation/hero/ai_gateway.png
---
KrakenD's AI Gateway capabilities extend your existing API infrastructure with controls to securely route, govern, and optimize Artificial Intelligence workloads at scale. The functionality offers a straightforward integration without extra layers, products, or complexity. It brings the speed, flexibility, and clarity you are already familiar with.

## What is an AI Gateway?
An AI Gateway is a specialized API Gateway designed to handle Artificial Intelligence workloads by providing secure, scalable, and manageable access to Large Language Models (LLMs) and other AI services. It acts as a control layer that routes AI-related traffic, enforces security and cost policies, and simplifies integration with multiple AI providers, helping developers manage AI services as part of their existing API ecosystem.

## Why do developers need AI control layers?
As AI adoption grows, so do concerns about security, governance, cost, and complexity. Developers need a dedicated control layer to:

- Protect sensitive AI data and enforce zero-trust security.
- Control and monitor token usage to avoid overspending.
- Manage multiple LLMs with unified, standardized configuration.
- Enforce governance policies such as rate limiting, prompt validation, and compliance checks.
- Simplify AI integration by abstracting vendor-specific APIs.

An AI Gateway ensures AI services are integrated reliably, securely, and cost-effectively within a company’s API infrastructure.

## Overview of KrakenD capabilities for AI gateway
To implement an AI Gateway with KrakenD, **you don't need to install an additional product** as everything is already bundled in KrakenD to serve dually as an API Gateway and AI Gateway.

KrakenD meets multiple operational demands when AI calls are involved, enabling you to secure sensitive data, transform data, control costs, enforce governance policies, and simplify development with minimal overhead. Without reinventing your infrastructure, you can keep your stack clean and scale and effectively adopt AI traffic.

The core functionality of KrakenD as an AI Gateway is classified under these four categories, each covering many use cases:

1. [Unified LLM interface](/docs/enterprise/ai-gateway/unified-llm-interface/) {{< badge >}}Enterprise{{< /badge >}}
2. [Security](/docs/v2.12/ai-gateway/security/)
3. [Cost Control](/docs/enterprise/ai-gateway/budget-control/) {{< badge >}}Enterprise{{< /badge >}}
4. [Governance](/docs/enterprise/ai-gateway/governance/) {{< badge >}}Enterprise{{< /badge >}}

See each section for the functionalities included.

### Unified LLM interface
Developers face complexity in integrating multiple LLM providers and managing proprietary APIs. KrakenD's AI Gateway simplifies this by providing a unified configuration that abstracts vendor-specific differences for API users. With standard AI API specs, easy protocol compatibility, and built-in telemetry endpoints, development teams can build once and deploy AI applications everywhere with less friction and overhead.

[Explore AI Gateway Unified LLM interface](/docs/v2.12/ai-gateway/unified-llm-interface/)

### AI Security
Protecting sensitive AI data and controlling access is essential for trustworthy AI workloads. KrakenD's AI Gateway integrates multiple layers of security at the edge to enforce **zero-trust AI operations**. From isolated authentication flows to data masking and exfiltration prevention, you can safeguard data traveling between clients and LLM providers without altering your existing API infrastructure.

[Explore AI Gateway Security functionality](/docs/v2.12/ai-gateway/security/)

### Cost Control
AI workloads can quickly generate unpredictable and excessive costs. The key to a successful adoption is being able to control expenses. KrakenD's AI Gateway provides granular token usage monitoring and enforcement to keep your AI expenses transparent and within budget.

[Explore AI Gateway Cost Control](/docs/enterprise/ai-gateway/budget-control/) {{< badge >}}Enterprise{{< /badge >}}

### Governance
Strong governance policies must be in place to deploy AI responsibly at scale. KrakenD's AI Gateway delivers comprehensive control via **multi-LLM routing**, prompt and response validation, rate limiting, and reusable prompt templates. These capabilities allow you to enforce compliance, security, and quality guardrails in line, reducing risk and standardizing AI interactions across teams or tenants.

[Explore AI Governance](/docs/enterprise/ai-gateway/governance/) {{< badge >}}Enterprise{{< /badge >}}


### Quickstart with KrakenD AI Gateway
Below is a simple example configuration snippet to enable AI Gateway capabilities in KrakenD. It includes just LLM routing, but as you can see above, there are many other components.

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
      "endpoint": "/llm/openai",
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
    }
  ]
}
```
There are many other possibilities; look at the rest of the AI sections.