---
lastmod: 2025-02-06
old_version: true
date: 2022-01-28
linktitle: Testing your plugins
title: "Test plugins after compiling them"
description: Learn how to test binary files that you compiled as plugins by loading them to KrakenD and test their behavior before you deploy them to production.
weight: 30
notoc: true
meta:
    since: v2.6
menu:
  community_v2.12:
    parent: "180 Extending with custom code"
---
Plugins are essential extensions to the KrakenD gateway, enhancing functionality without modifying the core codebase. Due to their reliance on specific versions of KrakenD, libraries, or system architecture, plugins can face compatibility issues following updates or modifications. So, when you have written a new plugin, and compiled it you still need to see that is loadable into KrakenD.

{{< note title="Recompile plugins when you upgrade KrakenD" type="warning" >}}
When you upgrade KrakenD to another version you must recompile your plugins using the [builder matching the version](/docs/v2.12/extending/writing-plugins/#plugin-builder).
{{< /note >}}

Even if a plugin compiles and passes initial tests on its own, it might fail when loaded into KrakenD. This could be due to various reasons such as compilation on a different architecture, mismatches in Go version, or discrepancies in library versions used by both the plugin and KrakenD.


The `test-plugin` command offers a real-scenario opportunity to test a compiled binary (usually a `.so` file) and verify if KrakenD can successfully load it.

## Testing a compiled plugin before it goes live
The `test-plugin` command requires you to pass the type of plugin you would like to test, and the path to the compiled binary.

The command accepts the following options:

{{< terminal title="Usage of test-plugin" >}}
{{< product test_plugin_command >}} -h
{{< ascii-logo >}}

Version: 2.12

Tests that one or more plugins are loadable into KrakenD.

Usage:
  {{< product test_plugin_command >}} [flags] [artifacts]

Examples:
{{< product test_plugin_command >}} -scm ./plugins/my_plugin.so ./plugins/my_other_plugin.so

Flags:
  -c, --client       The artifact should contain a Client Plugin.
  -h, --help         help for test
  -w, --middleware   The artifact should contain a Middleware Plugin.
  -m, --modifier     The artifact should contain a Req/Resp Modifier Plugin.
  -s, --server       The artifact should contain a Server Plugin.
{{< /terminal >}}

You can pass as many arguments as plugins you want to check at once. For instance, if you want to test that `plugin1.so` and `plugin2.so` are loadable as server plugins, you could execute `{{< product test_plugin_command >}} -s plugin1.so plugin2.so`.

You should provide at least on of the flags `-c` (client plugins), `-m` (req/resp modifiers), or `-s` (server plugins).

Here's an output example:

{{< terminal title="Checking a failing plugin example" >}}
{{< product test_plugin_command >}} -smc plugin1.so plugin2.so
[KO] SERVER	    plugin1.so: The plugin does not contain a HandlerRegisterer.
[KO] MODIFIER   plugin1.so: The plugin does not contain a ModifierRegisterer.
[OK] CLIENT     plugin1.so
[OK] CLIENT     plugin2.so
[OK] SERVER     plugin2.so
[OK] MODIFIER   plugin2.so
[KO] 2 tested plugin(s) in 13.498341ms.
1 plugin(s) failed.
{{< /terminal >}}

The command will exit with a status code `1` when it fails, so if you integrate it in a CI/CD pipeline it will stop.

When you find issues, use the `check-plugin` tool in development

The command exits with a **code of `1`** upon failure, allowing for integration into CI/CD pipelines to halt progress.

If you encounter issues, consider using the [check-plugin](/docs/v2.12/extending/check-plugin/) tool during development to diagnose and resolve problems effectively.