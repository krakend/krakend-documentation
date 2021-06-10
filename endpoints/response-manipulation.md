---
lastmod: 2019-01-31
date: 2016-09-30
toc: true
linktitle: Response manipulation
title: Response manipulation
weight: 10
menu:
  community_current:
    parent: "040 Endpoint Configuration"
images:
- /images/documentation/krakend-merge.png
---

KrakenD allows you to perform several manipulations of the responses out of the box, just by adding them to the configuration file. You can also add your own or 3rd parties middlewares to extend this behavior.

KrakenD manipulations are measured in `nanoseconds`, you can find the benchmark for every response manipulation in the [benchmarks](https://github.com/devopsfaith/krakend/blob/master/docs/BENCHMARKS.md#response-manipulation)

The following manipulations are available by default:

## Merging
When you create KrakenD endpoints, if a specific endpoint feeds from 2 or more backend sources (APIs), they will be automatically merged in a single response to the client. For instance, imagine you have 3 different API services exposing the resources `/a`,`/b`, and `/c` and you want to expose them all together in the KrakenD endpoint `/abc`. This is what you would get:

![Merge](/images/documentation/krakend-merge.png)

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

### Merging timeouts
Keep in mind that in order to avoid any degraded user experience, KrakenD won't be stuck forever until all the backends decide to respond. In a gateway **failing fast is better than succeeding slowly** and KrakenD will make sure this happens as it will **apply the timeout policy**. This will put your users safe during high load peaks, network errors, or any other problems that stress your backends.

The `timeout` value can be introduced inside each endpoint or globally placing `timeout` in the root of the configuration file. The most specific definition always overwrites the generic one.


#### What happens when the timeout is triggered or some backend failed?
If KrakenD is waiting for the backends to respond and the timeout is reached, the response will be incomplete and missing any data that couldn't be fetched before the timeout happened. On the other hand, all the parts that could effectively be retrieved before the timeout happened they will do appear in the response.

If the response has missing parts, the cache header won't exist, as we don't want clients to cache incomplete responses.

At all times, the `x-krakend-completed` header returned by KrakenD contains a boolean telling you if all backends returned its content (`x-krakend-completed: true`) or a partial response (`x-krakend-completed: false`).


### Merge example

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

## Filtering

When you create a KrakenD endpoint you can decide to show only a subset of the fields coming from the response of your backends.
There are a number of different reasons you might want to use this functionality, but we strongly encourage you to use it to save user's bandwidth and increase load and render times.

There are two different strategies you can use to filter content:

- **[Deny list](/docs/backends/data-manipulation#deny)**
- **[Allow list](/docs/backends/data-manipulation#allow)**

See [filtering documentation](/docs/backends/data-manipulation#filtering)

## Grouping
KrakenD is able to group your backend responses inside different objects. In other words, when you set a `group` attribute for a backend, instead of placing all the response attributes in the root of the response, KrakenD creates a new key and places the response inside.

Encapsulating backend responses inside each own group is especially interesting when different backend responses can have colliding key names (e.g: all responses contain an `id` with different values).

See [grouping documentation](/docs/backends/data-manipulation#grouping)

## Mapping (renaming)

KrakenD is also able to manipulate the name of the fields of the generated responses, so your composed response would be as close to your use case as possible without changing a line on any backend.

In the `mapping` section map the original field name with the desired name.

See [mapping documentation](/docs/backends/data-manipulation#mapping)

## Target (or capturing)
It is frequent in many API implementations that the important data is always encapsulated inside a generic field like `data`, `response` or `content`, as there are other fields showing the status code and other metadata. Sometimes we neither want to let the client handle with this nor drag this first level container through all the configuration.

When setting a `target` in your backend, these generic containers (the target) disappear and all content extracted to the root as it never existed. As this capturing takes place before other options like `allow` or `mapping`, you don't need to use nesting.

See [target documentation](/docs/backends/data-manipulation#target)

## Collection -or Array- manipulation
KrakenD expects all backends to return objects in the response. There are times when the whole response of the backend comes inside an array, and other times when you need to do operations over fields that are arrays themselves.

In any case, manipulations over arrays work differently than the objects.

See [collections documentation](/docs/backends/data-manipulation#collections)