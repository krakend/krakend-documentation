---
lastmod: 2024-11-27
date: 2021-11-25
linktitle: Extending KrakenD
title: Extending KrakenD with your code
description: Learn how to extend KrakenD API Gateway by developing custom plugins or scripts to add new functionalities and integrate with external systems
aliases: ["/docs/extending/introduction/"]
weight: -1
skip_header_image: true
menu:
  community_current:
    parent: "180 Extending with custom code"
images:
- /images/documentation/krakend-plugins.png
---

KrakenD is **highly extensible and flexible** and allows developers to extend its functionality through custom code when the built-in features are not enough. Whether you need to add custom logic, integrate specific business rules, or enhance features, KrakenD lets you add extensions coded by you. 

## Extending with Lua
[Lua](/docs/endpoints/lua/) is an embedded scripting language designed for simplicity and speed. It's perfect for **quick customizations**, such as:

- Request and response manipulation
- Custom validations and rules
- Dynamic transformations

### Lua advantages
- Simplicity: Lua is easy to learn and try.
- No Compilation: Changes are applied by editing the Lua script, making it faster to iterate and test.
- Runtime Flexibility: Scripts can be dynamically loaded and modified without restarting KrakenD.
- Ideal for Small Tasks: like header manipulation, simple data, transformations, or basic validation rules
- Portability: Lua scripts do not need modifications on KrakenD upgrades.

### Lua limitations
- Limited performance: Lua is interpreted, making it slower for CPU-intensive tasks.
- Lack of strong typing: Type safety and error handling are minimal, which could lead to runtime errors.
- No user-contributed libraries: You cannot import external libraries.
- Testing: Testing Lua scripts requires custom tooling or integration tests, as Lua doesn't have built-in testing frameworks akin to Go's tools.

## Extending with Go plugins
For more **advanced and performance-critical** requirements, KrakenD supports [plugins written in Go](/docs/extending/writing-plugins/). Using Go plugins ensures optimal performance for your extensions, and if you are fluent in Go, they are the best option for extensibility.

With Go plugins, you can pretty much do anything you want, including integrating with external services, using databases, and anything you can code. 


### Go plugins advantages
- High performance: Compiled Go plugins execute at native speed, suitable for heavy processing tasks.
- Extensive libraries: A world of Go libraries and an ecosystem for integrating with APIs, databases, and more.
- Strong typing: Compile-time checks reduce runtime errors, ensuring predictable behavior.
- Testable:
  - Write unit tests for your plugin logic using Go's testing framework.
  - Use CI/CD pipelines for automated testing and validation.
- Advanced capabilities with external system integration
- **Very** complex data manipulation

### Go plugins limitations
- Compilation Overhead: Each change requires recompilation, which is not suitable for "quick hacking"
- Deployment Complexity: Plugins are platform-specific (.so files), requiring recompilation for different OS/architecture setups. When you upgrade the KrakenD version, you need to recompile using the builder.
- KrakenD Restarts: Reloading the plugin requires restarting KrakenD.
- Higher Barrier to Entry: Requires Go expertise and familiarity with KrakenD's plugin contract.


## Lua or Go?
Both Lua and Go plugins allow you to extend KrakenD's capabilities, but their suitability depends on your use case, **team expertise** (this is key), and performance requirements. Summarizing:

- Lua is best for quick, simple, runtime modifications
- Go is best for complex, performance-critical, or testable extensions


{{< button-group >}}
{{< button url="/docs/endpoints/lua/" text="Get started with Lua" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
  <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 14.25v-2.625a3.375 3.375 0 0 0-3.375-3.375h-1.5A1.125 1.125 0 0 1 13.5 7.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H8.25m0 12.75h7.5m-7.5 3H12M10.5 2.25H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 0 0-9-9Z" />
</svg>

{{< /button >}}
{{< button url="/docs/extending/writing-plugins/" type="inversed" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6">
  <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 14.25v-2.625a3.375 3.375 0 0 0-3.375-3.375h-1.5A1.125 1.125 0 0 1 13.5 7.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H8.25m0 12.75h7.5m-7.5 3H12M10.5 2.25H5.625c-.621 0-1.125.504-1.125 1.125v17.25c0 .621.504 1.125 1.125 1.125h12.75c.621 0 1.125-.504 1.125-1.125V11.25a9 9 0 0 0-9-9Z" />
</svg> Get started with Go Plugins{{< /button >}}
{{< /button-group >}}


## What about forking?
Open-source users might be tempted to fork the source code to add modifications. Our recommended way to customize KrakenD is always through plugins or scripts, but you **should avoid forking the code** if you want to keep up to date with the product's progress and security vulnerabilities. We have seen over and over forked projects that are left behind because they don't have the resources to keep up.