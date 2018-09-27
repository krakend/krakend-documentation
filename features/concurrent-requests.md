---
lastmod: 2018-09-27
date: 2016-09-30
toc: true
linktitle: Concurrent Requests
title: Concurrent Requests
weight: 20
menu:
  documentation:
    parent: features
---
# Why enabling concurrent requests?
The concurrent requests is a way to ask for the same information at the same time to different backends, in order to **improve the response time and decrease error rates**.

Of course, this requires that your backend services run in multiple machines and are able to handle more load. When this is the case, you can set the configuration option `concurrent_calls` with a value bigger than 1 with all your idempotent requests.

# How does `concurrent_calls` work?
KrakenD will send up to N `concurrent_calls` to your backends for the same request. When the backends respond KrakenD will pick the fastest successful response ignoring any previous failures. Only in the case that all `concurrent_calls` fail, the user will receive also the failure.

The trade-off of this strategy is an increment of the load in the backend services, so make sure your infrastructure is ready for it. But your users will love it: Fewer errors, and faster responses!

To enable the concurrent requests in specific endpoints use the variable `concurrent_calls` in the `krakend.json` as follows:

	...,
	"endpoints": [
	{
	  "endpoint": "/products",
	  "method": "GET",
	  "concurrent_calls": "3",
	  ...

In the example configuration when a user calls the `/products` endpoint, KrakenD will open 3 different connections to the backends and will return the fastest successful response. The rest will be canceled.

## Impact of concurrent requests
In order to demonstrate the impact of this component, let's imagine two different scenarios: the happy one and the sad one. Here you have the cumulative distribution and the probability distribution of these two cases (the time range is just a 'placeholder' for whatever your actual response time values are, just replace the `100` with your `max_response_time`):

![CDF happy vs sad](/images/documentation/concurrency/CDF_happy_vs_sad.png)
![PDF happy vs sad](/images/documentation/concurrency/PDF_happy_vs_sad.png)

Just as a reminder: for the CDF graphs, the more to the left the line is, the better. For the PDF graphs, the more to the left and the narrow the peak is, the better.

This is the effect of using different concurrency levels for the happy case (notice how the response times are concentrated around 20% of the `max_response_time`):

![CDF concurrency happy](/images/documentation/concurrency/CDF_concurrency_happy.png)
![PDF concurrency happy](/images/documentation/concurrency/PDF_concurrency_happy.png)

Here, you have the effects for the sad case:

![CDF concurrency sad](/images/documentation/concurrency/CDF_concurrency_sad.png)
![PDF concurrency sad](/images/documentation/concurrency/PDF_concurrency_sad.png)

The concurrent request component also reduces the error rate of the exposed endpoint by **orders of magnitude**.

![Error concurrency up to 2](/images/documentation/concurrency/Error_concurrent_requests_up_2.png)
![Error concurrency bigger than 2](/images/documentation/concurrency/Error_concurrent_requests_bigger_2.png)

Since the scale of the previous graphs is hiding the huge impact, let's use a logarithmic scale:

![error concurrent calls](/images/documentation/concurrency/Error_concurrent_calls.png)
