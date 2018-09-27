---
lastmod: 2018-09-27
date: 2018-07-20
toc: true
linktitle:  Parameter forwarding
title: Parameter forwarding
weight: 30
menu:
  documentation:
    parent: features
---
KrakenD always **alleviates backends** avoiding to pollute them with everything coming from the clients. By default, **no parameters sent by clients are forwarded to backends** and if needed they will require an explicit declaration in the configuration file.

The parameter forwarding refers to:

- [Query string](#query-string-forwarding)
- [Headers](#headers-forwarding)
- [Cookies](#cookies-forwarding)

# Query string forwarding
Use `querystring_params`

In order to receive the query strings sent by clients in your backends, you will need to declare in the `querystring_params` the list of recognized parameters. When this list is set, any matching parameters are included in the backend call **when present**.

For instance, let's forward`?a=1&b=2` to the backends:

	{
	  "version": 2,
	  "endpoints": [
	    {
	      "endpoint": "/v1/foo",
	      "method": "GET",
	      "querystring_params": [
	        "a",
	        "b"
	      ],
	      "backend": [
	        {
	          "url_pattern": "/catalog",
	          "sd": "static",
	          "host": [
	            "http://some.api.com:9000"
	          ]
	        }
	      ]
	    }
	  ]
	}

With this configuration, given a request like `http://krakend:8080/v1/foo?a=1&b=2&c=3` the backend will receive `a` and `b` but `c` will be missing. But also, a request like `http://krakend:8080/v1/foo?a=1` the backend will receive `a` but `b` will be missing.

# Headers forwarding
Use `headers_to_pass`

Declare the list of headers sent by the client that you want to let pass to the backend with the `headers_to_pass` option.

A client request from a browser or a mobile client usually contains a lot of headers, including cookies. Typical examples of the kind of headers that are usually received from clients are `Host`, `Connection`, `Cache-Control`, `Cookie`... and a long long etcetera. The backend usually does not need any of this to return the content.

KrakenD will pass only the essential headers to the backends, for instance:

	Accept-Encoding: gzip
    Host: localhost:8080
    User-Agent: KrakenD Version {{% version %}}
    X-Forwarded-For: ::1

 When you use the `headers_to_pass` take into account that any of these headers will be replaced with the ones you declare.

 An example to pass the `User-Agent` to the backend:

	{
	  "version": 2,
	  "endpoints": [
	    {
	      "endpoint": "/v1/foo",
	      "method": "GET",
	      "headers_to_pass": [
        	"User-Agent"
      	  ],
	      "backend": [
	        {
	          "url_pattern": "/catalog",
	          "sd": "static",
	          "host": [
	            "http://some.api.com:9000"
	          ]
	        }
	      ]
	    }
	  ]
	}

This setting will change the headers received by the backend to:

	Accept-Encoding: gzip
    Host: localhost:8080
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36
    X-Forwarded-For: ::1

# Cookies forwarding
A cookie is just some content passing inside the `Cookie` header. If you want cookies to reach your backend, add the `Cookie` header under `headers_to_pass`, just as you would do with any other header. With this, **all your cookies** will be sent to all backends inside the endpoint. Use this option wisely!

Example:

	{
	  "version": 2,
	  "endpoints": [
	    {
	      "endpoint": "/v1/foo",
	      "method": "GET",
	      "headers_to_pass": [
        	"Cookie"
      	  ],
	      "backend": [
	        {
	          "url_pattern": "/catalog",
	          "sd": "static",
	          "host": [
	            "http://some.api.com:9000"
	          ]
	        }
	      ]
	    }
	  ]
	}
