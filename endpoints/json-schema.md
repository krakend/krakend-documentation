---
lastmod: 2021-11-17
date: 2020-07-10
linktitle: JSON Schema request validation
title: Validating the requests with JSON Schema
weight: 130
menu:
  community_current:
    parent: "040 Endpoint Configuration"
notoc: true
meta:
  since: 1.2
  source: https://github.com/krakendio/krakend-jsonschema
  namespace:
  - validation/json-schema
  scope:
  - endpoint
  - async_agent
---
KrakenD endpoints receiving a JSON object in its body can apply automatic validations using the [JSON Schema](https://json-schema.org/) vocabulary before the content passes to the backends. The json schema component allows you to define **validation rules** on the body, type definition, or even validate the fields' values.

When the validation fails, KrakenD returns to the user a status code `400` (Bad Request), and only if it succeeds, the backend receives the request.

## JSON Schema Configuration
The JSON Schema configuration has to be declared at the **endpoint level** with the namespace object `validation/json-schema`. KrakenD offers compatibility for the specs **draft-04, draft-06 and draft-07**.

The following example **checks if the body is a json object**:

```json
{
    "extra_config": {
        "validation/json-schema": {
            "type": "object"
        }
    }
}
```


You can apply constraints by adding keywords to the schema. For instance, you can check that the `type` is an instance of an object, array, string, number, boolean, or null.

All the configuration inside the namespace is pure JSON Schema vocabulary. [Read the JSON schema documentation](https://json-schema.org/) to get familiar with the specification.

 A full configuration for you to try on the localhost with the [debug endpoint](/docs/endpoints/debug-endpoint/) is:

```json
{
    "version": 3,
    "port": 8080,
    "host": [ "http://127.0.0.1:8080" ],
    "endpoints": [
        {
            "endpoint": "/address",
            "method": "POST",
            "backend": [
                {
                    "url_pattern": "/__debug/"
                }
            ],
            "extra_config":{
                "validation/json-schema": {
                  "type": "object",
                  "required": ["number", "street_name", "street_type"],
                  "properties": {
                    "number":      { "type": "number" },
                    "street_name": { "type": "string" },
                    "street_type": { "type": "string",
                                     "enum": ["Street", "Avenue", "Boulevard"]
                                   }
                  }
                }
            }
        }
    ]
}
```
Do you want to extend this example? try [this other example](https://json-schema.org/learn/examples/address.schema.json)

### Returning the error message
The default (and recommended) policy of KrakenD is to hide implementation details to the API consumers, and when a JSON schema fails, the gateway returns the `400` HTTP status code and no body.

Still, you can show the **JSON schema error message** to the end user by [enabling the `return_error_msg`](/docs/service-settings/router-options/#return_error_msg) in the router options.

The same example used above with the `return_error_msg` addition will output the problems when the schema does not validate:

```json
{
    "version": 3,
    "port": 8080,
    "host": [ "http://127.0.0.1:8080" ],
    "extra_config": {
        "router": {
           "return_error_msg": true
        }
    },
    "endpoints": [
        {
            "endpoint": "/address",
            "method": "POST",
            "backend": [
                {
                    "url_pattern": "/__debug/"
                }
            ],
            "extra_config":{
                "validation/json-schema": {
                  "type": "object",
                  "required": ["number", "street_name", "street_type"],
                  "properties": {
                    "number":      { "type": "number" },
                    "street_name": { "type": "string" },
                    "street_type": { "type": "string",
                                     "enum": ["Street", "Avenue", "Boulevard"]
                                   }
                  }
                }
            }
        }
    ]
}
```

And when calling it incorrectly:

{{< terminal title="Term" >}}
curl -i -X POST -d '{"invalid": true}' http://localhost:8080/address
HTTP/1.1 400 Bad Request
X-Krakend: Version {{< product latest_version >}}
X-Krakend-Completed: false
Date: Thu, 17 Nov 2022 08:57:53 GMT
Content-Length: 96
Content-Type: text/plain; charset=utf-8

- (root): number is required
- (root): street_name is required
- (root): street_type is required
{{< /terminal >}}
