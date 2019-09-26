---
lastmod: 2019-05-03
date: 2016-09-30
toc: true
linktitle: Data manipulation
title: Data manipulation
weight: 10
menu:
  documentation:
    parent: backends
---

# Filtering
When you offer a KrakenD endpoint you can decide to show only a subset of the fields coming from the backends' response or alter the structure of the content provided. There are a number of different reasons you might want to use this functionality, but we strongly encourage you to use it to save user's bandwidth and increase load and render times.

There are two different strategies you can use to filter content:

- **Blacklist**
- **Whitelist**

## Blacklist

The blacklist filter can be read as the *don't show this* filter. KrakenD will remove from the response all matching fields defined in the list, and the ones that do not match will be revealed. Use the blacklist to exclude some fields in the response.

To blacklist a field from the response add under the desired `endpoint` configuration an `blacklist` array with all the fields you don't want to show. E.g.:

    "blacklist": [
      "token",
      "CVV"
    ]

### Nested fields (dot operator)
The blacklisted fields of your choice can also be a nested field. Use a **dot** as the level separator. For instance the `a1` field in the following JSON response `{ "a": { "a1": 1 } }` can be blacklisted as `a.a1`.

Also worth mentioning, the operator works with **objects only** and not with arrays. When working with collections `[]`, see the special case [manipulating arrays](/docs/backends/flatmap/).

For instance, `{ "a": [ { "a1": 1 } ] }` can't be blacklisted as `a.a1` because `a1` is inside an array.

{{% note title="Working with arrays" %}}
Blacklist and whitelist filters expect objects. When working with collections, see the [flatmap approach](/docs/backends/flatmap/)
{{% /note %}}

### Blacklist example
We will use the [JSONPlaceholder](https://jsonplaceholder.typicode.com/) fake API so you can see live the output of the backend.

We want to set up a KrakenD endpoint that returns the **posts for a specific user**, but we've seen that the [backend response](https://jsonplaceholder.typicode.com/posts/1) contains too much data since our use case do not need the `body` and `userId` fields and we want a lighter and faster response.

The KrakenD endpoint to accept URLs like`/posts/1` is defined as follows:

	{
      "endpoint": "/posts/{user}",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/posts/{user}",
          "host": [
            "https://jsonplaceholder.typicode.com"
          ],
          "blacklist": [
            "body",
            "userId"
          ]
        }
      ]
    }

Now, when calling the KrakenD endpoint `/posts/1` the response you would get will be as follows:

    {
	  "id": 1,
	  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit"
	}

[Compared with the backend response](https://jsonplaceholder.typicode.com/posts/1) you'll see that the fields `body` and `userId` are no longer there.

## Whitelist
The blacklist filter can be read as the *only show this* filter. When you set a whitelist KrakenD will include in the endpoint response only those fields that match exactly with your choice. Use the whitelist to strictly define the fields you want to show in the response.

The whitelisted fields of your choice can also be nested field. Use a **dot** as the level separator. For instance the `a1` field in the following JSON response `{ "a": { "a1": 1 } }` can be whitelisted as `a.a1`.

### Whitelist example

We will repeat the same exercise we did in the blacklist to get the same output. We only want to get the `id` and `title` fields from the backend.

	{
      "endpoint": "/posts/{user}",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/posts/{user}",
          "host": [
            "https://jsonplaceholder.typicode.com"
          ],
          "whitelist": [
            "id",
            "title"
          ]
        }
      ]
    }

Now, when calling the KrakenD endpoint `/posts/1` the response you would get will be as follows:

    {
	  "id": 1,
	  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit"
	}

Just exactly as we did with the blacklist.

## Whitelist or blacklist?
When filtering you need to choose between the blacklist and the whitelist. The two operations cannot coexist as their behaviors are contradictory. Nothing stops you to play for a while and see what happens if you mix them, but make sure it's only experimentation!

From the performance point of view, blacklist is slightly fastest than whitelist.

# Grouping
KrakenD is able to group your backend responses inside different objects. In other words, when you set a `group` attribute for a backend, instead of placing all the response attributes in the root of the response, KrakenD creates a new key and places the response inside.

Encapsulating backend responses inside each own group is especially interesting when different backend responses can have colliding key names (e.g: all responses contain an `id` with different values).

When grouping different backend responses **don't share the same group name**, as the slowest backend would overwrite the response with the same group. Group names are supposed to be unique for each backend in the same endpoint but this is not enforced.

## Grouping example

The following code is an endpoint that gets data from two different backends but one of the responses is encapsulated inside the field `last_post`.

	{
      "endpoint": "/users/{user}",
      "method": "GET",
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
          ],
          "group": "last_post"
        }
      ]
    }

This will generate responses like this one:

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

# Mapping

Mapping, or also known as **renaming**, let you change the name of the fields of the generated responses, so your composed response would be as close to your use case as possible without changing a line on any backend.

In the `mapping` section map the original field name with the desired name.

## Mapping example:
Instead of showing the `email` field we want to name it `personal_email`:

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

Will generate responses like this one:

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

# Target
It is frequent in many API implementations that the desired data is always encapsulated inside a generic field like `data` or `content` and you don't want to have this level included in your responses.

Use capturing when you want to capture the content inside these generic containers and extract it to the root as it never existed and when you want to work with other manipulation options. Capture takes place before other options like whitelist or mapping.

The capturing option uses the attribute `target` in the configuration file.

## Capturing a target example
Given a backend endpoint with this kind of responses containing a level `data`:

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

and with this KrakenD configuration

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

the gateway will generate responses like this one:

	{
	    "updated":"2010-01-07T19:58:42.949Z",
	    "totalItems":800,
	    "startIndex":1,
	    "itemsPerPage":1,
	    "items":[]
	}

# Collections
Working with collections (or arrays) is a special manipulation case. There are two different scenarios regarding collections:

- When the whole backend response is inside an array instead of an object
- When you want to manipulate collections (e.g.: a path like `data.item[N].property`)

## When the Backend response is inside an array
KrakenD expects all backends to return an object as the response. For instance, a JSON response containing an object comes encapsulated in curly braces `{}`. E.g.:
```
{ "a": true, "b": false }
```

When your API does not return an object but a collection (`[]` or array) you need to declare it explicitly with `"is_collection": true` so that KrakenD can convert it to an object for further manipulation. An example of a JSON collection response is:

```
[    {"a": true },    {"b": false} ]
```
In such cases, add inside the `backend` key the property `"is_collection": true` so KrakenD can convert this collection to an object.

By default, KrakenD adds the key `collection` in the response, e.g.:
```
{
	"collection": [
		{"a": true },
		{"b": false}
	]
}
```
You can rename the default key name `collection` to something else using the `mapping` attribute (docs above, example below).

The following is a real example based on a [collection response](http://jsonplaceholder.typicode.com/posts), copy and paste to test in your environment:

```
"endpoints": [
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
]
```

The response will look like this:

```
{
  "myposts": [
    {
      ...
    },
    {
      ...
    }
}
```
## When you need to manipulate arrays
All the data manipulation operations (such as whitelist, blacklist, etc) expect to find objects in the response. When an object is nested inside another object you can filter directly, but when there are arrays in the equation KrakenD needs to flatten the structure.

See [flatmap documentation](/docs/backends/flatmap)

# Manipulations with Lua
Backends response can be transformed using Lua scripting.

See [Lua scripts](/docs/endpoints/lua/) documentation