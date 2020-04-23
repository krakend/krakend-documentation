---
lastmod: 2019-10-02
date: 2019-10-02
linktitle: Return backend errors
title: Returning the details of backend errors
weight: 130
#since: 0.9
#source: https://github.com/devopsfaith/krakend
menu:
  documentation:
    parent: backends
---

When you are willing to manipulate or aggregate data, KrakenD's policy regarding errors and status codes is to **hide from the client any backend details**. The philosophy behind this is that clients have to be decoupled from its underlying services.

If in the other hand, your endpoint connects to a single backend with no manipulation, [use the `no-op` encoding](/docs/endpoints/no-op) which returns the response to the client *as is*, preserving its form: body, headers, status codes and such.

You can override the default policy of hiding backend error details.

## Showing backend errors
If you prefer revealing these details to the client, you can choose to show them in the gateway response. To achieve this, enable the `return_error_details` option in backend the configuration, and then all errors will appear in the desired key.

Place the following configuration inside the `backend` configuration:

    "extra_config": {
        "github.com/devopsfaith/krakend/http": {
            "return_error_details": "backend_alias"
        }
    }

Notice that `return_error_details` sets an alias for this backend.

## Response for failing backends
When a backend fails, you'll find an object named `error_` + `backend_alias` containing the detailed errors of the backend. The returned structure on error contains the status code and the body:


	"error_backend_alias": {
		"http_status_code": 404,
		"http_body": "404 page not found\\n"
	}


If there are no errors, the key won't exist.

## Example
The following configuration sets an endpoint with two backends that return its errors in two different keys:

 	{
		"endpoint": "/detail_error",
		"backend": [
			{
				"host": ["http://127.0.0.1:8081"],
				"url_pattern": "/foo",
				"extra_config": {
					"github.com/devopsfaith/krakend/http": {
						"return_error_details": "backend_a"
					}
				}
			},
			{
				"host": ["http://127.0.0.1:8081"],
				"url_pattern": "/bar",
				"extra_config": {
					"github.com/devopsfaith/krakend/http": {
						"return_error_details": "backend_b"
					}
				}
			}
		]
    }

Let's say your `backend_b` has failed, but your `backend_a` worked just fine. The client response could look like this:

	{
		"error_backend_b": {
			"http_status_code": 404,
			"http_body": "404 page not found\\n"
		},
		"foo": 42
	}