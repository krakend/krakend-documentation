---
lastmod: 2021-05-02
date: 2020-07-10
linktitle: JSON Schema validation
title: Validating the body with the JSON Schema integration
weight: 130
menu:
  community_v1.3:
    parent: "040 Endpoint Configuration"
notoc: true
meta:
  since: 1.2
  source: https://github.com/devopsfaith/krakend-jsonschema
  namespace:
  - github.com/devopsfaith/krakend-jsonschema
  scope:
  - endpoint
---
KrakenD endpoints receiving a JSON object in its body can apply automatic validations using the [JSON Schema](https://json-schema.org/) vocabulary before the content passes to the backends. The json schema component allows you to define **validation rules** on the body, type definition, or even validate the fields' values.

When the validation fails, KrakenD returns to the user a status code `400` (Bad Request), and only if it succeeds, the backend receives the request. 

## JSON Schema Configuration
The JSON Schema configuration has to be declared at the **endpoint level** with the namespace object `github.com/devopsfaith/krakend-jsonschema`. For instance, to **check if the body is a json object**:

    "extra_config":{
        "github.com/devopsfaith/krakend-jsonschema": {
            "type": "object"
        }
    }

You can apply constraints by adding keywords to the schema. For instance, you can check that the `type` is an instance of an object, array, string, number, boolean, or null.

All the configuration inside the namespace is pure JSON Schema vocabulary. [Read the JSON schema documentation](https://json-schema.org/) to get familiar with the specification.

 A full configuration for you to try on the localhost with the [debug endpoint](/docs/v1.3/endpoints/debug-endpoint/) is:
{{< highlight json "hl_lines=14-25" >}}
{
    "version": 2,
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
                "github.com/devopsfaith/krakend-jsonschema": {
                  "type": "object",
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
{{< /highlight >}}
Do you want to extend this example? try [this other example](https://json-schema.org/learn/examples/address.schema.json)
