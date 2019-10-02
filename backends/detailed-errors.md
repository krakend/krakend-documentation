---
lastmod: 2019-10-02
date: 2019-10-02
linktitle: Return backend errors
title: Returning the details of backend errors
weight: 130
#since: 0.9
#source: https://github.com/devopsfaith/krakend
notoc: true
menu:
  documentation:
    parent: backends
---

When you are willing to manipulate or aggregate data, KrakenD's policy regarding errors and status codes is to **hide from the client any backend details**. The philosophy behind this is that clients have to be decoupled from its underlying services.

If in the other hand, your endpoint connects to a single backend with no manipulation, [use the `no-op` encoding](/docs/endpoints/no-op) which returns the response to the client *as is*, preserving its form: body, headers, status codes and such.

You can override the default policy of hiding backend error details. If you prefer revealing these details to the client, you can choose to show them in the gateway response. To achieve this, enable the `return_error_details` option in backend the configuration, and then all errors will appear in the desired key.

Place the following configuration inside the `backend` configuration:

    "extra_config": {
        "github.com/devopsfaith/krakend/http": {
            "return_error_details": "object_containing_error_details"
        }
    }

Notice that the `return_error_details` key requires a value that represents the name of the object containing the error details. If there are no errors, the key won't exist.

# Example
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
				"url_pattern": "/var",
				"extra_config": {
					"github.com/devopsfaith/krakend/http": {
						"return_error_details": "backend_b"
					}
				}
			}
		]
    }
