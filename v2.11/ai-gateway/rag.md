---
lastmod: 2025-05-21
old_version: true
date: 2025-05-21
linktitle: RAG Pipeline
title: AI Security
description: Protect sensitive data and secure access to LLMs using KrakenD’s API-level AI security capabilities like isolated auth and data masking.
menu:
  community_v2.11:
    parent: "100 AI Gateway"
weight: 100

---
## RAG Pipelines (DRAFT)
You can use KrakenD as a secure, lightweight proxy layer in a **RAG (Retrieval-Augmented Generation)** pipeline, and even enforce exfiltration prevention and data sanitization on the fly.

As per today, the usage of RAG requires you to make manual use of the [sequential proxy](/docs/v2.11/endpoints/sequential-proxy/) and implement the flow in the configuration.

A typical RAG pipeline looks like this:

1. User prompt
2. API Gateway (KrakenD)
3. Retriever (e.g., vector DB)
4. Enricher / Processor (e.g., metadata enrichers, relevance filters)
5. LLM
6. Response to user (with or without additional data manipulation)

See the [Sequential Proxy](/docs/v2.11/endpoints/sequential-proxy/) and [Workflow](/docs/enterprise/endpoints/workflows/#content) components to implement this logic.

This component is valid both for:

1. Training the LLM
2. Query model