---
lastmod: 2021-12-11
old_version: true
date: 2021-12-11
linktitle: Run integration tests
title: Automated integration tests
notoc: true
menu:
  community_v2.5:
    parent: "170 API Documentation and Dev Tools"
weight: 30
---
In addition to checking the syntax of your KrakenD configuration and make sure that the gateway can start, you can run **integration tests** to make sure that the gateway returns the expected content from the consumed backends. to make sure all endpoints are properly connected and that they reply with the expected content. To do that, you can use the library that KrakenD is relying on to run its **integration tests**, and complement the unit testing battery.

KrakenD comes with a small program that lets you define a folder with tests and execute them all at once, reporting any failures it found.

The way this library works is quite simple. You create a folder with all the different specs you want to tested, one per file. Each file is a `.json` file with two keys:

- `in`: The parameters used to build the request to a running KrakenD with your configuration
	- `method`: The request method
	- `url`: The full URL to the endpoint you want to test
	- `header`: An optional map of header to include in the request
 	- `body`: An optional payload you can send in the request as data
- `out`: The expected response from the gateway
	- `status_code` (*integer*): The expected status code
	- `body`: The returned body by the response as a string, or as JSON object.
	- `header`: Any header included in the response. When the value is empty it means that you don't expect that header.

For instance:

```json
{
	"in": {
		"method": "GET",
		"url": "http://localhost:8080/detail_error",
		"header": {
			"X-Header": "something"
		}
	},
	"out": {
		"status_code": 200,
		"body": {
          "message": "pong"
        },
		"header": {
			"content-type": ["application/json; charset=utf-8"],
			"Cache-Control": [""],
			"X-Krakend-Completed": ["true"]
		}
	}
}
```

In the example above, the response must contain the `content-type` and `X-Krakend-Completed` with the specified values, and the `Cache-Control` cannot be present.

You must build the go binary that allows you to run the tests.

## Installing the integration tests tool
To install the integration tests you only need to run the following lines in any machine or Docker container with go installed:

{{< terminal title="Installing the integration tool" >}}
go install github.com/krakend/krakend-ce/v2/cmd/krakend-integration@v2.5
{{< /terminal >}}

After this you should have a new binary `krakend-integration` in your PATH. To use it you will to execute:

{{< terminal title="Term" >}}
krakend-integration -krakend_bin_path ./krakend \
-krakend_config_path ./krakend.json \
-krakend_specs_path ./specs
{{< /terminal >}}

The three parameters of the binary are:

- `-krakend_bin_path`: The path to the KrakenD binary. Defaults to `./krakend`, but if KrakenD is in your PATH then you can use `krakend`.
- `-krakend_config_path`: The path to the KrakenD configuration file that contains all endpoints you want to test.
- `-krakend_specs_path`: The path to the folder containing all your specifications. These are all the tests that you want to run against KrakenD and check its output.


{{< button-group >}}
{{< button url="https://github.com/krakend/krakend-ce/tree/master/tests" text="Integration tests" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 3v4M3 5h4M6 17v4m-2-2h4m5-16l2.286 6.857L21 12l-5.714 2.143L13 21l-2.286-6.857L5 12l5.714-2.143L13 3z" />
</svg>
{{< /button >}}
{{< button url="https://github.com/krakend/krakend-ce/tree/master/tests/fixtures" text="Fixtures" type="inversed" >}}<svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
</svg>{{< /button >}}
{{< /button-group >}}
