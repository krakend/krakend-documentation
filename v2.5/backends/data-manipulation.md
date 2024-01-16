---
lastmod: 2020-07-24
old_version: true
date: 2016-09-30
toc: true
linktitle: "Basic data manipulation: Group, Filter, Map"
title: Data Manipulation in KrakenD API Gateway
description: Explore data manipulation capabilities in KrakenD API Gateway, enabling transformation and manipulation of response data
weight: 20
menu:
  community_v2.5:
    parent: "060 Request and Response Manipulation"
meta:
  noop_incompatible: true
---
This page describes the most basic options to manipulate the content you receive from the `backend` before delivering it to the endpoint to [aggregate data from all backends](/docs/v2.5/endpoints/response-manipulation/#aggregation-and-merging).

<!--more-->
{{< note title="Before you begin..." type="tip" >}}
See what kind of response you are getting from your backend in the first place. If you get the response inside an object `{}`, you can apply the manipulations right away. But if you get an array `[]`, see how to manipulate **collections** below.
{{< /note >}}

## Filtering
When you offer a KrakenD endpoint, you can decide whether to return all the fields from the backend (default behavior) or specify which ones are allowed through an allow or deny list. You might want to use this functionality for many different reasons. Still, we strongly encourage you to consider **using it frequently to save the user's bandwidth**, provide the client what is needed, and decrease the load and render times.

You can use two different filtering strategies, pick one or the other in each endpoint:

- **Deny list** (`deny`)
- **Allow list** (`allow`)


**Note**: Prior to KrakenD 1.2 these terms where known with the outdated term `whitelist` and `blacklist`.


### Deny

The deny list filter can be read as the *don't show this* filter. KrakenD will remove from the backend response all matching fields (case-sensitive) defined in the list, and the ones that do not match are returned. Use the deny list to exclude some fields in the response.

To exclude a field from the response, add under the desired `backend` configuration a `deny` array with all the fields you don't want to show. E.g.:

```json
{
    "deny": [
      "token",
      "CVV",
      "password"
    ]
}
```

#### Nested fields (dot operator)
The deny fields of your choice can also be nested ones. Use a **dot** as the level separator. For instance the `a1` field in the following JSON response `{ "a": { "a1": 1 } }` can be added in the deny list as `a.a1`.

Also worth mentioning, the operator works with **objects only** and not with arrays. When working with collections `[]`, see the special case [manipulating arrays](/docs/v2.5/backends/flatmap/).

For instance, `{ "a": [ { "a1": 1 } ] }` does not work as `a.a1` because `a1` is inside an array, but `{ "a": { "a1": 1 } }` would work.

{{< note title="Working with arrays" >}}
Deny list and allow list filters expect objects. When working with collections, see the [flatmap approach](/docs/v2.5/backends/flatmap/)
{{< /note >}}

#### Deny list example
We will use the [JSONPlaceholder](https://jsonplaceholder.typicode.com/) fake API to see live the output of the backend.

We want to set up a KrakenD endpoint that returns the **posts for a specific user**, but we've seen that the [backend response](https://jsonplaceholder.typicode.com/posts/1) contains too much data since our use case do not need the `body` and `userId` fields and we want a lighter and faster response.

The KrakenD endpoint to accept URLs like`/posts/1` is defined as follows:

```json
{
  "endpoint": "/posts/{user}",
  "method": "GET",
  "backend": [
    {
      "url_pattern": "/posts/{user}",
      "host": [
        "https://jsonplaceholder.typicode.com"
      ],
      "deny": [
        "body",
        "userId"
      ]
    }
  ]
}
```



When calling the KrakenD endpoint `/posts/1` the response you would get will be as follows:

```json
{
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit"
}
```

[Compared with the backend response](https://jsonplaceholder.typicode.com/posts/1), you'll see that the fields `body` and `userId` are no longer there.

### Allow
The allow list filter can be read as the *only show this* filter. When you set an allow list KrakenD will include in the endpoint response, only those fields that match exactly with your choice. Use the allow list to strictly define the fields you want to show in the response.

The allowed fields of your choice can also be nested. Use a **dot** as the level separator. For instance the `a1` field in the following JSON response `{ "a": { "a1": 1 } }` can be defined as `a.a1`.

#### Allow list example

We will repeat the same exercise we did in the deny list to get the same output. We only want to get the `id` and `title` fields from the backend.

```json
{
  "endpoint": "/posts/{user}",
  "method": "GET",
  "backend": [{
    "url_pattern": "/posts/{user}",
    "host": [
      "https://jsonplaceholder.typicode.com"
    ],
    "allow": [
      "id",
      "title"
    ]
  }]
}
```


When calling the KrakenD endpoint `/posts/1` the response you would get will be as follows:

```json
{
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit"
}
```

Just exactly as we did with the deny list.

### Allow list or deny list?
When filtering, you need to choose between the deny list and the allow list. The two operations cannot coexist as their behaviors are contradictory. Nothing stops you from playing for a while and seeing what happens if you mix them, but make sure it's only experimentation!

Strictly from a performance point of view, the deny list is slightly fastest than the allow list.

## Grouping
KrakenD can group your backend responses inside different objects. When you set a `group` attribute for a backend, instead of placing all the response attributes in the root of the response, KrakenD creates a new key and encapsulates the response inside.

Encapsulating backend responses inside each own group is especially interesting when different backend responses can have colliding key names (e.g., all responses contain an `id` with different values).

When grouping different backend responses **don't share the same group name**, as the slowest backend would overwrite the response with the same group. Group names are supposed to be unique for each backend in the same endpoint but this is not enforced.

### Grouping example

The following code is an endpoint that gets data from two different backends, but one of the responses is encapsulated inside the field `last_post`.
{{< highlight json "hl_lines=12" >}}
{
    "endpoint": "/users/{user}",
    "method": "GET",
    "backend": [
      {
        "url_pattern": "/users/{user}",
        "host": ["https://jsonplaceholder.typicode.com"]
      },
      {
        "url_pattern": "/posts/{user}",
        "host": ["https://jsonplaceholder.typicode.com"],
        "group": "last_post"
      }
    ]
  }
{{< /highlight >}}

This will generate responses like this one:
{{< highlight json "hl_lines=23-28">}}
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
  "last_post": {
    "id": 1,
    "userId": 1,
    "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
    "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
  }
}
{{< /highlight >}}

## Mapping

Mapping, or also known as **renaming**, let you change the name of the fields of the generated responses, so your composed response would be as close to your use case as possible without changing a line on any backend.

In the `mapping` section, map the original field name with the desired name.

### Mapping example:
Instead of showing the `email` field we want to name it `personal_email`:

```json
{
  "endpoint": "/users/{user}",
  "method": "GET",
  "backend": [
    {
      "url_pattern": "/users/{user}",
      "host": [
        "https://jsonplaceholder.typicode.com"
      ],
      "mapping": {
        "email": "personal_email"
      }
    }
  ]
}
```

Will generate responses like this one:

{{< highlight json "hl_lines=5">}}
{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "personal_email": "Sincere@april.biz",
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
{{< /highlight >}}

## Target
It is frequent in many API implementations that the desired data lives inside a generic field like `data` or `content`, and you don't want to have the encapsulating level included in your responses.

Use `target` when you want to capture the content inside these generic containers and work with the rest of the manipulation options as if it were on the root. The capture takes place before other options like allow list or mapping.

The capturing option uses the attribute `target` in the configuration file.

### Capturing a target example
Given a backend endpoint with this kind of responses containing a level `data`:

{{< highlight json "hl_lines=3" >}}
{
  "apiVersion":"2.0",
  "data": {
    "updated":"2010-01-07T19:58:42.949Z",
    "totalItems":800,
    "startIndex":1,
    "itemsPerPage":1,
    "items":[]
  }
}
{{< /highlight >}}

And we want a response with the contents inside `data` only, like:

```json
{
  "updated":"2010-01-07T19:58:42.949Z",
  "totalItems":800,
  "startIndex":1,
  "itemsPerPage":1,
  "items":[]
}
```

We need this KrakenD configuration:

{{< highlight json "hl_lines=7" >}}
{
    "endpoint": "/foo",
    "method": "GET",
    "backend": [
      {
        "url_pattern": "/bar",
        "target": "data"
      }
    ]
  }
{{< /highlight >}}

## Moving, extracting, or flattening
There will be times when you want to move one item to another place, or flatten its structure. In these cases you need to use the [flatmap component](/docs/v2.5/backends/flatmap/).

For instance, you have a nested response like the following:

```json
{
    "shipping_id": "f15f8c62-8c63-46de-a7f6-a08f131848c5",
    "zone": {
        "state": "NY",
        "zip": "10001"
    }
}
```
And you would like to **extract out its fields**, like this:

```json
{
    "shipping_id": "f15f8c62-8c63-46de-a7f6-a08f131848c5",
    "shipping_state": "NY",
    "shipping_zip": "10001"
}
```

You would need a configuration like this:

```json
{
  "backend": [{
      "url_pattern": "/shipping",
      "extra_config": {
          "proxy": {
              "flatmap_filter": [
                  { "type": "move", "args": ["zone.state","shipping_state"] },
                  { "type": "move", "args": ["zone.zip","shipping_zip"] },
                  { "type": "del","args": ["zone"] }
              ]
          }
      }
  }]
}
```
See how the [flatmap filter](/docs/v2.5/backends/flatmap/) works for more options.

## Collections
Working with collections (or arrays) is a special manipulation case. There are different scenarios regarding collections:

- When the whole backend response is inside an array instead of an object
- When you want to manipulate collections (e.g., a path like `data.item[N].property`)

### When the Backend response is inside an array
KrakenD expects all backends to return an object as the response as the default behavior. For instance, a JSON response containing an object comes encapsulated in curly braces `{}`. E.g.:

```json
{ "a": true, "b": false }
```


When your API does not return an object but a collection (`[]` or array) you need to declare it explicitly with `"is_collection": true` so that KrakenD can convert it to an object for further manipulation. An example of a JSON collection response is:

```json
[ {"a": true }, {"b": false} ]
```

In such cases, add inside the `backend` key the property `"is_collection": true` so KrakenD can convert this collection to an object.

{{< note title="Automatic detection of arrays" type="note" >}}
The use of `is_collection` can be avoided when the backend uses as `encoding` the value `safejson`. See [supported encodings](/docs/v2.5/backends/supported-encodings/)
{{< /note >}}

By default, KrakenD adds the key `collection` in the response, e.g.:

```json
{
  "collection": [
    {"a": true },
    {"b": false}
  ]
}
```

You can rename the default key name `collection` to something else using the `mapping` attribute (docs above, the example below), but you can also **return directly the array** to the end-user, without any wrapping, if the endpoint has `"output_encoding": "json-collection"`.

The following is a real example based on a [collection response](http://jsonplaceholder.typicode.com/posts), copy and paste to test in your environment:

{{< highlight json "hl_lines=8-11">}}
{
  "endpoint": "/posts",
  "backend": [
    {
      "url_pattern": "/posts",
      "host": ["http://jsonplaceholder.typicode.com"],
      "sd": "static",
      "is_collection": true,
      "mapping": {
        "collection": "myposts"
      }
    }
  ]
}
{{< /highlight >}}

The response will look like this:

```json
{
  "myposts": [
    {  },
    {  }
  ]
}
```

### When you need to manipulate arrays
All the data manipulation operations (such as the allow list, deny list, etc.) expect to find objects in the response `{}`. When there are arrays instead, KrakenD needs a special configuration that internally flattens this structure:

See [Manipulating arrays - flatmap](/docs/v2.5/backends/flatmap/)

## More advanced manipulations
If you need more sophisticated manipulation options, there are different approaches you can use:

- Through a [Query Language](/docs/enterprise/endpoints/jmespath/) {{< badge >}}Enterprise{{< /badge >}}
- Through a [template](/docs/enterprise/backends/response-body-generator/) {{< badge >}}Enterprise{{< /badge >}}
- Through [Response modifier plugins](/docs/v2.5/extending/plugin-modifiers/) - Very performant, requires compilation
- Through [Lua scripting](/docs/v2.5/endpoints/lua/) - Less performant, does not require compilation
