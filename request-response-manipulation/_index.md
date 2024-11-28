---
lastmod: 2024-11-26
date: 2024-11-26
toc: true
linktitle: Manipulation toolkit
title: Introduction to Request and Response Manipulation
description: Explore the request and response manipulation capabilities in KrakenD API Gateway, allowing you to modify and enhance API responses for a better user experience
weight: -1
menu:
  community_current:
    parent: "060 Request and Response Manipulation"
images:
  - /images/documentation/diagrams/req-resp-manipulation-intro.mmd.svg
---

One of KrakenD's standout features is its powerful capability to manipulate requests and responses, allowing you to shape data flows to meet your application's needs. With KrakenD, you can transform incoming requests and outgoing responses on the fly, ensuring your APIs are efficient and perfectly aligned with your business logic. Whether you need to filter sensitive information, restructure payloads, enrich responses, or enforce specific rules, KrakenD provides a robust toolkit to get the job done.

Before you start digging into manipulation, identify which part of the request or response needs modification and define the transformation logic. Then, choose the right components from the range of declarative configurations to suit different manipulation tasks. There are several components at your disposal that we could group as follows:

- **Request and response transformation**: Functionality to rewrite paths, filter parameters, or inject new data based on specific logic, adjust responses before they reach the client, add or remove fields, aggregate data, or change formats to match client expectations.
- **Protocol mapping and performance**: Translate and reformat payload structures, ensuring automatic encoding conversion from APIs with differing formats. Employ cache or gzipped responses to increase delivery speed.
- **Request and response validation**: Although not a manipulation itself, it allows you to abort requests or responses that do not have compliant data.


## Data transformation and aggregation
KrakenD dynamically reshapes, filters, and transforms requests and responses, ensuring compatibility between clients and backends. This simplifies payloads, improves performance, and customizes data delivery to fit client needs. The components available to transform data are:

- [API Composition and Aggregation](/docs/endpoints/response-manipulation/): Combine multiple API responses into a unified payload.
- [Response Manipulation](/docs/backends/data-manipulation/): The basic operators allow you to group, filter, map, or capture responses.
- [Static Manipulation](/docs/backends/martian/): Modify requests and responses with static data. For instance, hardcode authentication on an API, or add new headers to a request.
- [Manipulation of collections](/docs/backends/flatmap/): Handle and manipulate arrays sequentially in payloads.

In addition, the {{< badge >}}Enterprise{{< /badge >}} edition allows you to set complex logic:

- [Response Manipulation with a Query Language](/docs/enterprise/endpoints/jmespath/): Dynamically manipulate complex payloads with advanced query expressions.
- [Response Manipulation with Regexp](/docs/enterprise/endpoints/content-replacer/): Replace and mask content using regular expressions.
- [Response Manipulation with Templates](/docs/enterprise/backends/response-body-generator/): Write a template defining the response delivered to the client
- [Request Manipulation with Templates](/docs/enterprise/backends/body-generator/): Write a template defining the request sent to the backend
- [Global Response Header modification](/docs/enterprise/service-settings/response-headers-modifier/): Globally alter the headers returned to clients.
- [Request Enrichment with GeoIP](/docs/enterprise/endpoints/geoip/): Enrich requests with geolocation data for tailored functionality.
- [Workflows](/docs/enterprise/endpoints/workflows/): Sometimes you cannot do all the transformations in a single shot, and a Workflow can solve that.


## Protocol mapping and performance
KrakenD transparently converts data from one format to another and even transforms protocols. It ensures smooth integration with various systems and optimizes payload sizes for better efficiency. Some functionalities covering this are:

- [Interpretation of backend responses](/docs/backends/supported-encodings/): Tell the gateway what type of response it will read, and it will be able to manipulate its data automatically.
- [Re-encoding of responses](/docs/endpoints/content-types/): Choose the format of responses returned to clients. It does not matter their origin, whether JSON, XML, SOAP, etc.
- [Response Caching](/docs/backends/caching/): Improve performance by caching frequently used responses.
- [Non-rest connectivity](/docs/enterprise/non-rest-connectivity/): Connect to Lambda functions, GraphQL, Pub/Sub, and others.

On {{< badge >}}Enterprise{{< /badge >}}:

- [SOAP](/docs/enterprise/backends/soap/): Craft the body and XML content you will send to a SOAP service and treat it back as XML or JSON automatically.
- [Gzip Compression](/docs/enterprise/service-settings/gzip/): Optimize payload size with Gzip compression
- [Static server](/docs/enterprise/endpoints/serve-static-content/): Use the gateway as a static web server

## Request and response validation
In addition to transforming data and protocols, you can also validate requests and responses against JSON Schema definitions or enforce size restrictions to maintain data integrity. These features ensure data compliance, protect against oversized payloads and enhance system reliability.

- [JSON Schema Request Validation](/docs/endpoints/json-schema/): Validate that the payload set by the client conforms to a schema.
- [Conditional requests and responses ](/docs/endpoints/common-expression-language-cel/): Allow the request to go in or out based on business rules
- [Static responses on errors](/docs/endpoints/static-proxy/): Add static content to complement the final response on incomplete and degraded responses.

Plus on the {{< badge >}}Enterprise{{< /badge >}}:

- [JSON Schema Response Validation](/docs/enterprise/endpoints/response-schema-validator/): Validate backend responses according to a schema before delivering them to the client.
- [Setting Maximum Request Size](/docs/enterprise/endpoints/maximum-request-size/): Control the maximum size of the payload received
    

## Why manipulate requests and responses?
Modern APIs must often adapt to varying client requirements, integrate with legacy systems, or comply with stringent data security and performance standards. In addition, most of the time, implementors of these APIs do not have control over the origin of the data. Request and response manipulation allows you to:

- **Optimize payloads**: Remove unnecessary data or transform structures to enhance performance. Sending fewer bytes through the network increases the user experience and conversion rates.
- **Enforce data standards**: Standardize payload formats, validate fields, or inject default values.
- **Enhance security**: Mask or strip sensitive information before delivering it to clients or backends.
- **Support dynamic use cases**: Apply custom transformations tailored to different user roles or device types.
