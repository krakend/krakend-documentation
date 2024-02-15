---
lastmod: 2023-03-13
old_version: true
date: 2023-03-13
description: The Backend For Frontend pattern allows you to create an API consumption layer that provides aggregated "views" of several services.
linktitle: Backend For Frontend
title: The Backend For Frontend pattern
weight: 10
menu:
  community_v2.3:
    parent: "170 Design principles"
---
One of the main differences between KrakenD and any other API gateway in the market is that all your endpoints can offer a ready-to-use Backend For Frontend experience by simply declaring the backend sources and the fields your client needs. But why and when do you need to use a BFF?

## The problem a BFF solves
When developers of APIs create new endpoints, whether in a microservices pattern or a monolith, using an intermediate gateway or directly connecting to backends, they must provide **general-purpose responses**. The responses from the backends, or upstream services, must cover all intended cases around the domain they belong to.

For instance, as a backend developer, if you create a `/user` endpoint, it should return **all the user data for the different consumers** of the API. Of course, some consumers will use more significant portions of the data than others, but generally speaking, most clients will be parsing unnecessary data and wasting bandwidth:

![Problem 1 of API usage](/images/documentation/diagrams/backend-for-frontend-problem.mmd.svg)

Another problem is that the clients consuming this data, like a desktop web UI, a mobile app, a TV app, etc., have very different capabilities and needs. Even data consumption between Android and iOS native apps is different!

In addition to the problems above, each client needs to rely on **multiple domains** to complete a single use case. For instance, on the front page screen of an e-commerce application (whether it's a mobile app or web page), you might need to simultaneously consume data from the `user`, `catalog`, `cart`, `pricing`, and `promotions` services.


![Problem with multiple API usage](/images/documentation/diagrams/backend-for-frontend-problem-2.mmd.svg)

As you can see in the diagram above, the client must deal with four different HTTP connections, wait for the fat responses coming back by a slow channel (network), parse all the data, and pick only the attributes needed to render the page. So why wouldn't you give what the client needs in the first place, omitting unnecessary data?

Finally, upstream services are developed simultaneously with the UI in most cases, adding new capacities. As a result, the backend becomes a general-purpose service, serving the requirements of all the desktop, mobile, or IoT interfaces.

Desktop and mobile interfaces usually compete in requirements, and the backends keep growing to fit both types of usage. Every frontend team will work in different data views to display the content, and the backend team becomes a bottleneck as it must fulfill the frontend team's needs. In many cases, the same type of information (or view) will require different response formats and response errors for each consumer, making your backend developers spend a lot of time developing this logic. And many meetings and time making the different frontend dev teams get on the contract agreement!

## How the BFF works
KrakenD implements the BFF pattern with aggregation, allowing your frontend teams to define precisely the information you want to retrieve from the different services and how to deal with the errors.

![BFF example](/images/documentation/diagrams/backend-for-frontend.mmd.svg)

The client receives:

- Exactly the data it needs, and no more.
- The information for a use case is taken from a single call
- Aggregated information containing all involved services
- Optionally, stub static data when there's missing information
- Separation of concerns
- Decoupling: Do not have to worry very much about the backend changing or evolving

### Configuring a BFF
To add data from multiple services, you only need to declare [new backends](/docs/v2.3/backends/) when creating endpoints. The backends can be REST services, but you can use services from different places, like lambdas, queues, gRPC, etc.

Then you can choose which attributes you want to include in the response using [filtering strategies](/docs/v2.3/backends/data-manipulation/#filtering), and if needed, add manipulation. In most cases, you will want each service [grouped](/docs/v2.3/backends/data-manipulation/#grouping) in a new object, so you have each service in a different entity.

The gateway will automatically aggregate all the data for you and let the client know using the `X-KrakenD-Completed` header when all the information has been retrieved. Additionally, you can return [stub data when there are errors](/docs/v2.3/endpoints/static-proxy/).

For instance, the following use case of a front page, would retrieve only a few fields from the two services it consume:

```json
{
  "endpoints": [
    {
      "endpoint": "/frontpage",
      "method": "GET",
      "backend": [
        {
          "url_pattern": "/user/{id_user}",
          "method": "GET",
          "host": ["http://user-service"],
          "allow": ["id_user", "name", "email"],
          "group": "user"
        },
        {
          "url_pattern": "/catalog",
          "method": "GET",
          "host": ["http://catalog-service"],
          "allow": ["featured"]
        }
      ]
    }
  ]
}
```
The response of the `/frontpage` endpoint could look like this:

```json
{
  "user": {
    "id_user": 123,
    "name": "John Doe",
    "email": "john@doe.com"
  },
  "featured": {
    "today": {},
    "week": {}
  }
}
```
## When to use this pattern

The Backend for Frontend pattern in KrakenD allows you to have one API per user interface or different endpoints per user interface. Then you can define how the data is returned to each interface, both when there are errors or in the happy path, without needing to get on an agreement with other frontend teams.

Use the BFF when you want or need:

- To call multiple backend services to serve a use case
- Less chatty responses and retrieving what is needed only
- Avoid over-processing data
- Transform or manipulate data before returning it to the client
- Optimize the bandwidth usage and network calls number
- Implement a Facade pattern
- Have a front app with simpler code and less maintenance
- Reduce backend development overhead
- Save time getting to team agreements

KrakenD allows powerful transformations and business logic, but we recommend leaving the gateway to do basic gluing and avoid having complex logic complicated to maintain.
