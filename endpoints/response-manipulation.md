---
aliases:
- /docs/features/response-manipulation/
lastmod: 2019-01-31
date: 2016-09-30
toc: true
linktitle: Response manipulation
title: Response manipulation
weight: 10
menu:
  documentation:
    parent: endpoints
---

KrakenD allows you to perform several manipulations of the responses out of the box, just by adding them to the configuration file. You can also add your own or 3rd parties middlewares to extend this behavior.

KrakenD manipulations are measured in `nanoseconds`, you can find the benchmark for every response manipulation in the [benchmarks](https://github.com/devopsfaith/krakend/blob/master/docs/BENCHMARKS.md#response-manipulation)

The following manipulations are available by default:

# Merging
When you create KrakenD endpoints, if a specific endpoint feeds from 2 or more backend sources (APIs), they will be automatically merged in a single response to the client. For instance, imagine you have 3 different API services exposing the resources `/a`,`/b`, and `/c` and you want to expose them all together in the KrakenD endpoint `/abc`. This is what you would get:

<img class="img-fluid" src="/images/documentation/krakend-merge.png" />

The merge operation is implemented in a way where user experience and responsiveness goes first. It does its *best effort* to get all the required parts from the involved backends and returning the composed object as soon as possible.

By simply adding several backends into an endpoint, you get the merge operation automatically.

The configuration for the image above could be like this:


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

## Merging timeouts
Keep in mind that in order to avoid any degraded user experience, KrakenD won't be stuck forever until all the backends decide to respond. In a gateway **failing fast is better than succeeding slowly** and KrakenD will make sure this happens as it will **apply the timeout policy**. This will put your users safe during high load peaks, network errors, or any other problems that stress your backends.

The `timeout` value can be introduced inside each endpoint or globally placing `timeout` in the root of the configuration file. The most specific definition always overwrites the generic one.


### What happens when the timeout is triggered or some backend failed?
If KrakenD is waiting for the backends to respond and the timeout is reached, the response will be incomplete and missing any data that couldn't be fetched before the timeout happened. On the other hand, all the parts that could effectively be retrieved before the timeout happened they will do appear in the response.

If the response has missing parts, the cache header won't exist, as we don't want clients to cache incomplete responses.

At all times, the `x-krakend-completed` header returned by KrakenD contains a boolean telling you if all backends returned its content (`x-krakend-completed: true`) or a partial response (`x-krakend-completed: false`).


## Merge example

Imagine an endpoint with the following configuration:

	...
	"endpoints": [
        {
	      "endpoint": "/users/{user}",
	      "method": "GET",
	      "timeout": "800ms"
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

When a user calls the endpoint `/users/1`, KrakenD will send two requests and, in the happy scenario, it will receive these responses:

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

and

	{
	  "userId": 1,
	  "id": 1,
	  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
	  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
	}

With these 'partial responses' and the given configuration, KrakenD will return the following response:

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

# Filtering

When you create a KrakenD endpoint you can decide to show only a subset of the fields coming from the response of your backends. There are a number of different reasons you might want to use this functionality, but we strongly encourage you to use it to save user's bandwidth and increase load and render times.

There are two different strategies you can use to filter content:

- Blacklist
- Whitelist

## Blacklisting fields (don't show)

The blacklist filter can be read as the *don't show this* filter. KrakenD will remove from the response all matching fields defined in the list, and the ones that do not match will be revealed. Use the blacklist to exclude some fields in the response.

The blacklisted fields of your choice can also be nested field. Use a **dot** as the level separator. For instance the `a1` field in the following JSON response `{ "a": { "a1": 1 } }` can be blacklisted as `a.a1`.

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

## Whitelisting fields (only show)
The blacklist filter can be read as the *only show this* filter. When you set a whitelist KrakenD will include in the endpoint response only those fields that match exactly with your choice. Use the whitelist to strictly define the fields you want to show in the response.

The whitelisted fields of your choice can also be nested field. Use a **dot** as the level separator. For instance the `a1` field in the following JSON response `{ "a": { "a1": 1 } }` can be whitelisted as `a.a1`.


### Whitelist example
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

## Whitelist or blacklist
When filtering you need to choose the blacklist or the whitelist, as the behaviors are contradictory and can't coexist. Nothing stops you to play for a while and see what happens if you mix them, but you've been warned!



# Grouping
KrakenD is able to group partial responses from backends under a user-defined field. In other words, when you set the `group` attribute for a backend, instead of placing all the response attributes in the root, KrakenD will create a new key and place the response inside.

Grouping is useful to encapsulate every backend call under different keys, especially when different backend responses can have colliding key names (e.g: all responses contain an `id` with different values).

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

# Mapping (renaming)

KrakenD is also able to manipulate the name of the fields of the generated responses, so your composed response would be as close to your use case as possible without changing a line on any backend.

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

# Capturing
It is frequent in many API implementations that the desired data is always encapsulated inside a generic field like `data` or `content` and you don't want to have this level in your responses.

Use capturing when you want to capture the content inside these generic containers and extract it to the root as it never existed and when you want to work with other manipulation options. Capture takes place before other options like whitelist or mapping.

The capturing option uses the attribute `target` in the configuration file.

## Capturing example
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

# Collections or Arrays
KrakenD expects that all backends return an object. For instance, a JSON response needs to be encapsulated between curly braces `{}`. E.g.:

```
{
	"a": true,
	"b": false
}
```

When your API returns a collection (`[]` or array) instead of an object you need to tell this to KrakenD so its content can be converted to an object. Example of collection:

```
[
	{"a": true },
	{"b": false}
]
```
In these cases, add inside the `backend` key the property `"is_collection": true` so KrakenD can convert this collection to an object. By default, the key `collection` will be added, e.g.:
```
{
	"collection": [
		{"a": true },
		{"b": false}
	]
}
```
You can rename the automatically named key `collection` to something else using the `mapping` attribute (see above).

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
{{% note title="No manipulation on arrays" %}}
When working with backend responses that are collections be aware that
data manipulations are not available (except for the mapping/renaming of the
`collection` key)
