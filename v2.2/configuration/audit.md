---
lastmod: 2023-01-31
old_version: true
date: 2023-01-18
linktitle: Configuration audit
title: Security audit of your configuration
description: The krakend audit command evaluates a configuration file's integrity and prints security recommendations and statistical information.
weight: 21
notoc: false
images:
- /images/documentation/screenshots/krakend_audit.png
menu:
  community_v2.2:
    parent: "010 Configuration file(s)"
---

The `krakend audit` command is a rule evaluation tool that checks configuration files written in any of its [supported formats](/docs/v2.2/configuration/supported-formats/) and returns practical **security recommendations**. It is designed to raise basic red flags and provide essential advice on your configuration. The output of the configuration and classification is inspired by the [CIS Benchmarks](https://www.cisecurity.org/communities/benchmarks).


{{< note title="Security disclaimer" type="warning" >}}
If the audit command passes, it does not mean that your API is necessarily secure but that the evaluated rules have passed (find them as recommendations below).
{{< /note >}}

The tool displays practical and helpful information, including (but not limited to) misconfigurations opening the door to vulnerabilities, presence/absence of key components, dangerous flags or combinations, conflicting declarations, and statistics (planned), to put a few examples.

## Audit vs Check
The `audit` command is complementary to the [`check` command](/docs/v2.2/configuration/check/). Still, instead of focusing on the configuration file structure and linting, it evaluates the logic at a different stage. The command executes after parsing the configuration, using a **summarized tree** of the final recognized components and flags loaded. It **does not has access to the values of the properties** (but `check` does). For instance, if you write a `jwk_url` to validate tokens, `audit` does not know if you are using HTTPS or HTTP or which domain, but it does see if you have `disable_jwk_security`, which is dangerous in production.

The purpose of the audit command is to add extra checks in your [automated CI pipeline](/docs/v2.2/deploying/ci-cd/) to have safer deployments.

## Audit configuration
The `audit` command has the following options:

{{< terminal title="Usage of KrakenD audit" >}}
krakend audit --help
{{< ascii-logo >}}

Version: 2.2

Audits a KrakenD configuration.

Usage:
  krakend audit [flags]

Examples:
krakend audit -i 1.1.1,1.1.2 -s CRITICAL -c krakend.json

Flags:
  -c, --config string        Path to the configuration file
  -f, --format string        Inline go template to render the results (default "{{ range .Recommendations }}{{.Rule}}\t[{{colored .Severity}}]   \t{{.Message}}\n{{ end }}")
  -h, --help                 help for audit
  -i, --ignore string        List of rules to ignore (comma-separated, no spaces)
  -I, --ignore-file string   Path to a text-plain file containing the list of rules to exclude
  -s, --severity string      List of severities to include (comma-separated, no spaces) (default "CRITICAL,HIGH,MEDIUM,LOW")
{{< /terminal >}}

The simplest version of the command requires the path to the configuration file only, and outputs any problems found:

{{< terminal title="Audit configuration" >}}
krakend audit -c krakend.json
1.2.1	[HIGH]   	 Prioritize using JWT for endpoint authorization to ensure security.
2.2.1	[MEDIUM]   Hide the version banner in runtime.
3.3.4	[CRITICAL] Set timeouts to below 1 minute for improved performance.
5.2.3	[LOW]   	 Avoid coupling clients by overusing no-op encoding.
{{< /terminal >}}

It also accepts different flags to customize its behavior:

- `--severity`, `-s`: Severity requirements
- `--ignore`, `-i`: Rules to ignore (inline)
- `--ignore-file`, `-I`: Rules to ignore (from file)
- `--format`, `--f`: Format of the output (Go template)

More details of the flags below.

### Configuring the audit severity
By default, the audit command will include **all severity levels**. However, you can choose through the `--severity` flag which groups you want to be printed in the console, separated by commas. The list is:

- `CRITICAL`
- `HIGH`
- `MEDIUM`
- `LOW`

When the `--severity` is not defined, KrakenD uses `--severity CRITICAL,HIGH,MEDIUM,LOW`. You can use a **comma separated** string (no spaces) with all the severities you want to print. For instance, using the same example we had above, to filter by the most severe problems you would type:

{{< terminal title="Term" >}}
krakend audit --severity CRITICAL,HIGH -c krakend.json
1.2.1	[HIGH]   	 Prioritize using JWT for endpoint authorization to ensure security.
3.3.4	[CRITICAL] Set timeouts to below 1 minute for improved performance.
{{< /terminal >}}

#### Excluding security audit rules
The outputted security recommendations by the command are generic to any installation and might not apply to your setup, or you might disagree with our assigned severity. You can exclude future checking of any specific audit rules by passing a list or creating an exception file. To do that, use the `--ignore` flag passing a comma-separated list (no spaces) with all the ignore rules or a `--ignore-file` with the path to an ignore file.

**All rules must have the numeric format `x.y.z`**.

For the inline option, you could do the following:

{{< terminal title="Ignore rules 1.2.3 and 4.5.6" >}}
krakend audit --ignore=1.2.3,4.5.6 -c krakend.json
{{< /terminal >}}

For the option of an ignore file, you should create a plain text file with one rule per line. You can place this file anywhere and it does not require a specific extension or name. However, if it is not in the KrakenD workdir (`/etc/krakend/`), you must specify its relative or absolute path:

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
Finally, you can choose the **format of the output** according to your needs by injecting a Go template using the `-f` flag. The flag expects an inline template.

The default template, as shown in the screenshot, applies the following go template:

```go-text-template
{{ range .Recommendations }}{{.Rule}}\t[{{colored .Severity}}]\t{{.Message}}\n{{ end }}
```
As the ouput is processed using a template, you can inject anything you like. For instance, the example below generates a [TOML file](https://toml.io/en/) into `recommendations.toml`.

{{< terminal title="Custom output" >}}
krakend audit -f '{{range .Recommendations}}
[[recommendation]]
  rule = "{{.Rule}}"
  message = "{{.Message}}"
  severity = "{{.Severity}}"
{{end}}' > recommendations.toml
{{< /terminal >}}

Or the **JSON format** is even easier to write:

{{< terminal title="JSON output" >}}
krakend audit -f '{{ marshal . }}' > recommendations.json
{{< /terminal >}}

As you can see, the templates use a series of **variables and functions**, as follows:

- `.Recommendations`: An array with all the recommendations, where each recommendation has the following structure:
    - `.Rule`: The identifier of this rule. E.g., `1.1.1`
    - `.Message`: A short message describing the recommendation.
    - `.Severity`: The level of severity for this recommendation

In addition, you can use two functions:

- `colored` to add colors to the severity (`{{colored .Severity}}`) when the output is in a terminal
- `marshal` to return a JSON representation of the variable. E.g., `{{ marshal . }}`

<!--  -`.Stats` -->

## Audit recommendations
The following is the list of **recommendations** you can find in the audit results. The recommendations are classified using a numeric code with the format `x.y.z`. You can use this rule identifier to exclude the rules during its checking, as explained above.

{{% audit_rules %}}