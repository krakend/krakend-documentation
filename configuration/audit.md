---
lastmod: 2023-01-18
date: 2023-01-18
linktitle: Security audit
title: Auditing the security of the configuration
description: The krakend audit command evaluates a configuration file's integrity and prints security recommendations and statistical information.
weight: 30
notoc: false
menu:
  community_current:
    parent: "010 Configuration file(s)"
---

The `krakend audit` command **evaluates the integrity of KrakenD configuration files** written in any of its [supported formats](/docs/configuration/supported-formats/), and returns **security recommendations**, but also other practical and helpful information, including (but not limited to) misconfigurations opening the door to vulnerabilities, presence/absence of key components, dangerous flags or combinations, conflicting declarations, and statistics to put a few examples.

The output of the configuration and classification is inspired by the [CIS Benchmarks](https://www.cisecurity.org/communities/benchmarks).

The `audit` command is complementary to the [`check` command](/docs/configuration/check/). Still, instead of focusing on the configuration file structure and linting, it evaluates the logic at a different stage. The command executes after parsing the configuration, using a **summarized tree** of the final recognized components and flags loaded. It **does not has access to the values of the properties**. For instance, if you write a `jwk_url` to validate tokens, it does not know if you are using HTTPS or HTTP or which domain, but it does see if you have `disable_jwk_security`, which is dangerous in production.

The purpose of the audit command is to add extra checks in your [automated CI pipeline](/docs/deploying/ci-cd/) to have safer deployments.

## Audit configuration
The simplest version of the command requires the configuration file only and shows the detected problems with your configuration:

{{< terminal title="Audit configuration" >}}
krakend audit -c krakend.json
[HIGH] 1.1.1: Ensure that Basic Auth is not used
[MEDIUM] 1.1.2: Ensure to protect your endpoints with stateless authorization methods such as JWT. Avoid API keys.
[HIGH] 1.2.1: Ensure to protect your endpoints with authorization, preferably with JWT
{{< /terminal >}}

It also accepts different flags to customize its behavior:

- `--severity`: Severity requirements
- `--ignore`: Rules to ignore (inline)
- `--ignore-file`: Rules to ignore (from file)
- `--f`: Format of the output (Go template)

More details of the flags below:

### Configuring the audit severity
By default the audit command will include **all severity levels**, although you can choose through the `--severity` flag which levels you want to be printed in the console, separated by commas. The list is:

- `CRITICAL`
- `HIGH`
- `MEDIUM`
- `LOW`

When the `--severity` is not defined, KrakenD uses `--severity CRITICAL,HIGH,MEDIUM,LOW`. You can use a **comma separated** string (no spaces) with all the severities you want to print. For instance, to see only the most severe problems you would type:

{{< terminal title="Term" >}}
krakend audit --severity CRITICAL,HIGH -c krakend.json
{{< /terminal >}}

#### Excluding security audit rules
The printed security recommendations by the command are generic to any installation and might not apply to your setup, or you might disagree with our assigned severity. You can exclude checking any specific audit rules by passing the list or creating an exception file. To do that, use the `--ignore` flag passing a comma-separated list (no spaces) with all the ignore rules or a `--ignore-file` with the path to an ignore file.

For the inline option, you could do the following:

{{< terminal title="Ignore rules 1.2.3 and 4.5.6" >}}
krakend audit --ignore=1.2.3,4.5.6 -c krakend.json
{{< /terminal >}}

For the option of an ignore file, you should create a plain text file with one rule per line. You can place this file anywhere. However, if it is not in the KrakenD workdir (`/etc/krakend/`), you must specify its relative or absolute path:

{{< terminal title="Content of the ignore file" >}}
cat .audit_ignore
1.2.3
4.5.6
{{< /terminal >}}

And then calling it with:
{{< terminal title="Ignore rules 1.2.3 and 4.5.6" >}}
krakend audit --ignore-file=.audit_ignore -c krakend.json
{{< /terminal >}}

### Customizing the output
Finally, you can choose the **format of the output** according to your needs by injecting a Go template using the `-f` flag. The flag expects an inline template. For instance:

{{< terminal title="Custom output" >}}
krakend audit -f '{{range .recommendations}}
[[recommendation]]
  rule = "{{.rule}}"
  title = "{{.title}}"
  severity = "{{.severity}}"
{{end}}' > recommendations.toml
{{< /terminal >}}

The example above generates a `recommendations.toml` file with a very different format. You could format it like JSON or any other format.

The variables available in the custom template are:

- `.recommendations`: An array with all the recommendations, where each recommendation has the following structure:
    - `.rule`: The identifier of this rule. E.g., `1.1.1`
    - `.title`: The title describing this rule.
    - `.severity`: The level of severity for this recommendation
- `.stats`

## Audit Rules
The following is the list of **recommendations** you can find in the audit results. The recommendations are classified using a numeric code with the format `x.y.z`. You can use this rule identifier to exclude future checking, as explained above.

{{% audit_rules %}}