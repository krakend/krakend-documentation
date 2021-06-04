---
lastmod: 2021-05-02
date: 2018-11-11
toc: true
linktitle: Lambda functions
title: Integration with AWS Lambda functions
weight: 110
images:
    - /images/krakend-lambda.png
menu:
  community_current:
    parent: "050 Backends Configuration "
notoc: true
meta:
  since: 1.0
  source: https://github.com/devopsfaith/krakend-lambda
  namespace:
  - github.com/devopsfaith/krakend-lambda
  scope:
  - backend
---

The Lambda integration allows you to **invoke Amazon Lambda functions** on a KrakenD endpoint call. The content returned by the lambda function can be treated and manipulated as any other backend.

The **payload** that is sent to the Lambda function comes from the request and depends on the method used by the `endpoint`:

*   Method `GET`: The payload contains all the parameters of the request.
*   Non-`GET` methods: The payload is defined by the content of the **body** in the request.

You don't need to set an Amazon API Gateway in the middle as KrakenD does this job for you.

## Lambda configuration

The inclusion requires you to add the code in the `extra_config` of your `backend` section, using the `github.com/devopsfaith/krakend-lambda` namespace.

The supported parameters are:

*   `function_name`: Name of the lambda function as saved in the AWS service.
*   `function_param_name`: The endpoint `{placeholder}` that sets the function name. You have to choose between `function_name` and `function_param_name` but not both.
*   `region`: The AWS identifier region (e.g.: `us-east-1`, `eu-west-2`, etc. )
*   `max_retries`: Maximum times you want to execute the function until you have a successful response.
*   `endpoint`: An optional parameter to customize the Lambda endpoint to call. Useful when Localstack is used for testing.

{{< note title="Parameters' first character uppercased" >}}
Notice the capitalization of the first letter of the parameter names at the configuration in the examples below. For instance, when an endpoint parameter is defined as `{route}`, define it in the config as `Route`.
{{< /note >}}


### Authentication

The KrakenD machine needs to have the AWS credentials in the default file, `~/.aws/credentials`.

When setting the credentials make sure that the lambda is callable within the KrakenD box with the credentials provided. This translates in having an IAM user with a policy and execution role that let you invoke the function.

## Example: Associate a lambda to a backend

When the KrakenD endpoint is attached to the same Lambda, use this configuration:

    "backend": [
    {
        "github.com/devopsfaith/krakend-lambda": {
            "function_name": "lambda-function",
            "region": "us-west1",
            "max_retries": 1
        }
    }

## Example: Take the lambda from the URL

When the name of the Lambda to depends on a parameter passed in the endpoint, use this configuration instead:

    "endpoint": "/call-a-lambda/{lambda},
    "backend": [
    {
        "github.com/devopsfaith/krakend-lambda": {
            "function_param_name": "Lambda",
            "region": "us-west1",
            "max_retries": 1
        }
    }

In this example, the `function_param_name` is telling us which is the placeholder in the endpoint setting the lambda. In this case, `{lambda}`.

Following the code, a call `GET /call-a-lambda/my-lambda` would produce a call to the `My-lambda` function in AWS.
