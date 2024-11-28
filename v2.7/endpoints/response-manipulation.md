---
lastmod: 2022-09-30
old_version: true
date: 2016-09-30
toc: true
linktitle: API Composition and aggregation
title: API Composition and aggregation
description: Explore the response manipulation capabilities in KrakenD API Gateway, allowing you to modify and enhance API responses for a better user experience
weight: 10
menu:
  community_v2.7:
    parent: "060 Request and Response Manipulation"
images:
- /images/krakend-merge.png
meta:
  noop_incompatible: true
---

KrakenD allows you to perform several manipulations of the responses out of the box by adding them to the configuration file. You can also add your own or 3rd parties middleware to extend this behavior.

KrakenD performance tests measure the operations in `nanoseconds`, and you can find the benchmark for every response manipulation in the [benchmarks section](https://github.com/luraproject/lura/blob/master/docs/BENCHMARKS.md#response-manipulation)

The following manipulations are available by default:

## Aggregation and merging
When you have more than one `backend` connected to an `endpoint` that **is not** using the `no-op` encoding, the gateway **aggregates and merges** the responses from all backends automatically in the final response.

For instance, imagine you have three different API services exposing the resources `/a`,`/b`, and `/c`, and you want to disclose them all together in the KrakenD endpoint `/abc`. This is what you would get:

![Merge](/images/krakend-merge.png)

The merge operation chooses user experience and responsiveness first. It makes its *best effort* to get all the necessary parts from the involved backends and return the composed object as soon as possible.

KrakenD marks the result of the merging operation with the `X-KrakenD-Completed` header, being `true` if all backends succeeded or `false` if some failed. When none succeeded, the gateway returns a `500` status code to the user.

The configuration for the image above could be like this:

```json
{
    "endpoints": [
        {
            "endpoint": "/abc",
            "timeout": "800ms",
            "method": "GET",
            "backend": [
                {
                    "url_pattern": "/a",
                    "encoding": "json",
                    "host": [
                        "http://service-a.company.com"
                    ]
                },
                {
                    "url_pattern": "/b",
                    "encoding": "xml",
                    "host": [
                        "http://service-b.company.com"
                    ]
                },
                {
                    "url_pattern": "/c",
                    "encoding": "json",
                    "host": [
                        "http://service-c.company.com"
                    ]
                }
            ]
        }
    ]
}
```

### Merging timeouts
Keep in mind that to avoid any degraded user experience, KrakenD won't be stuck forever until all the backends decide to respond. In a gateway **failing fast is better than succeeding slowly**, and KrakenD will make sure this happens as it will **apply the timeout policy**. It will protect your users during high load peaks, network errors, or other problems that stress your backends.

The `timeout` value can be introduced inside each endpoint or globally, placing `timeout` at the root of the configuration file. The most specific definition always overwrites the generic one.

#### What happens when the timeout is triggered, or some backend fails?
If KrakenD waits for the backends to respond and the timeout is reached, the response will be incomplete and missing any data it couldn't fetch before the timeout happened. On the other hand, all the parts that the gateway could effectively retrieve before the timeout occurred will appear in the response.

If the response has missing parts, the cache header won't exist, as we don't want clients to cache incomplete responses.

At all times, the `X-Krakend-Completed` header contains a boolean telling you if all backends returned their content (`x-krakend-completed: true`) or it's a partial response (`x-krakend-completed: false`).


### Merge example

Imagine an endpoint with the following configuration:

```json
{
    "endpoints": [
        {
            "endpoint": "/users/{user}",
            "method": "GET",
            "timeout": "800ms",
            "backend": [
                {
                    "url_pattern": "/users/{user}",
                    "host": [
                        "https://jsonplaceholder.typicode.com"
                    ]
                },
                {
                    "url_pattern": "/posts/{user}",
                    "host": [
                        "https://jsonplaceholder.typicode.com"
                    ]
                }
            ]
        }
    ]
}
```



When a user calls the endpoint `/users/1`, KrakenD will send two requests and, in the happy scenario, it will receive these responses:

```json
{
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
        "street": "Kulas Light",
        "suite": "Apt. 556",
        "city": "Gwenborough",
        "zipcode": "92998-3874",
        "geo": {
            "lat": "-37.3159",
            "lng": "81.1496"
        }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
        "name": "Romaguera-Crona",
        "catchPhrase": "Multi-layered client-server neural-net",
        "bs": "harness real-time e-markets"
    }
}
```


and

```json
{
    "userId": 1,
    "id": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```


With these 'partial responses' and the given configuration, KrakenD will return the following response:

```json
{
    "id": 1,
    "name": "Leanne Graham",
    "username": "Bret",
    "email": "Sincere@april.biz",
    "address": {
        "street": "Kulas Light",
        "suite": "Apt. 556",
        "city": "Gwenborough",
        "zipcode": "92998-3874",
        "geo": {
            "lat": "-37.3159",
            "lng": "81.1496"
        }
    },
    "phone": "1-770-736-8031 x56442",
    "website": "hildegard.org",
    "company": {
        "name": "Romaguera-Crona",
        "catchPhrase": "Multi-layered client-server neural-net",
        "bs": "harness real-time e-markets"
    },
    "userId": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```


## Filtering

When you create an endpoint, you can choose to show only a subset of the fields coming from the response of your backends.
You might want to use this functionality for several reasons, but we strongly encourage you to use it to save users' bandwidth and decrease load and render times.

There are two different strategies you can use to filter content:

- **[Deny list](/docs/v2.7/backends/data-manipulation/#deny)**
- **[Allow list](/docs/v2.7/backends/data-manipulation/#allow)**

See [filtering documentation](/docs/v2.7/backends/data-manipulation/#filtering)

## Grouping
You can group (or encapsulate or wrap) your backend responses inside different objects. In other words, when you set a `group` attribute for a backend, instead of placing all the response attributes in the root of the response, KrakenD creates a new key and places the response inside.

Encapsulating backend responses inside each group is interesting when different backend responses can have colliding key names (e.g: all responses contain an `id` with different values).

When you consume aggregated content, use the `group` strategy.

See [grouping documentation](/docs/v2.7/backends/data-manipulation/#grouping)

## Mapping (renaming)

KrakenD can also manipulate the name of the fields of the generated responses, so your composing response would be as close to your use case as possible without changing a line on any backend.

In the `mapping` section, map the original field name with the desired name.

See [mapping documentation](/docs/v2.7/backends/data-manipulation/#mapping)

## Target (or capturing)
It is frequent in many API implementations that the vital data is always encapsulated inside a generic field like `data`, `response`, or `content`, along with other fields showing the status code and metadata. Sometimes we neither want to let the client handle this nor drag this first-level container through all the configurations.

When setting a `target` in your backend, these generic containers (the target) disappear, and all content is extracted to the root as if it never existed. As this capturing takes place before other options like `allow` or `mapping`, you don't need to use nesting.

See [target documentation](/docs/v2.7/backends/data-manipulation/#target)

## Collection -or Array- manipulation
KrakenD expects all backends to return objects in the response. There are times when the whole response of the backend comes inside an array, and other times when you need to do operations over fields that are arrays themselves.

In any case, manipulations over arrays work differently than the objects.

See [collections documentation](/docs/v2.7/backends/data-manipulation/#collections)

## Query language manipulation
The most powerful option to manipulate content without scripts or plugins is via the JMESpath component, which allows you to apply changes using a JSON query language.

For example: *return the name of all the students older than 18*:
```js
students[?age > `18` ].name
```

See [Advanced query language manipulation {{< badge color="denim">}}Enterprise{{< /badge >}}](/docs/enterprise/endpoints/jmespath/)

## Content replacement with regular expressions
Another Enterprise feature is the Content Replacer plugin, which allows you to search objects in the responses and apply regular expression replacements, or literal replacements.

For instance:

```json
    {
        "payment.credit_card": {
            "@comment": "Ridiculous card masking. Take 4 digits and remove the rest. Credit card is nested inside the payment object.",
            "find": "(^\\d{4})(.*)",
            "replace": "${1}-XXXX",
            "regexp": true
        }
    }
```

See [Content Replacer {{< badge >}}Enterprise{{< /badge >}}](/docs/enterprise/endpoints/content-replacer/)