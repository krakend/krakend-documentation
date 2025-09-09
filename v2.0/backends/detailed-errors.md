---
lastmod: 2021-05-02
old_version: true
date: 2019-10-02
linktitle: Return backend errors
title: Returning the details of backend errors
weight: 130
menu:
  community_v2.0:
    parent: "050 Backends Configuration"
meta:
  since: v0.8
  source: https://github.com/luraproject/lura
  namespace:
  - backend/http
  scope:
  - backend
---

When you are willing to manipulate or aggregate data, KrakenD's policy regarding errors and status codes is to **hide from the client any backend details**. The philosophy behind this is that clients have to be decoupled from its underlying services.

If in the other hand, your endpoint connects to a single backend with no manipulation, [use the `no-op` encoding](/docs/v2.0/endpoints/no-op/) which returns the response to the client *as is*, preserving its form: body, headers, status codes and such.

You can override the default policy of hiding backend error details.

## Showing backend errors
If you prefer revealing these details to the client, you can choose to show them in the gateway response. To achieve this, enable the `return_error_details` option in backend the configuration, and then all errors will appear in the desired key.

Place the following configuration inside the `backend` configuration:

{{< highlight json >}}
{
	"url_pattern": "/return-my-errors",
	"extra_config": {
        "backend/http": {
            "return_error_details": "backend_alias"
        }
    }
}
{{< /highlight >}}

Notice that `return_error_details` sets an alias for this backend.

## Response for failing backends
When a backend fails, you'll find an object named `error_` + its `backend_alias` containing the detailed errors of the backend. The returned structure on error contains the status code and the body:

{{< highlight json >}}
{
	"error_backend_alias": {
		"http_status_code": 404,
		"http_body": "404 page not found\\n"
	}
}
{{< /highlight >}}




If there are no errors, the key won't exist.

## Example
The following configuration sets an endpoint with two backends that return its errors in two different keys:

{{< highlight json >}}
{
		"endpoint": "/detail_error",
		"backend": [
			{
				"host": ["http://127.0.0.1:8081"],
				"url_pattern": "/foo",
				"extra_config": {
					"backend/http": {
						"return_error_details": "backend_a"
					}
				}
			},
			{
				"host": ["http://127.0.0.1:8081"],
				"url_pattern": "/bar",
				"extra_config": {
					"backend/http": {
						"return_error_details": "backend_b"
					}
				}
			}
		]
    }
{{< /highlight >}}

Let's say your `backend_b` has failed, but your `backend_a` worked just fine. The client response could look like this:

{{< highlight json >}}
{
	"error_backend_b": {
		"http_status_code": 404,
		"http_body": "404 page not found\\n"
	},
	"foo": 42
}
{{< /highlight >}}
