---
lastmod: 2018-11-27
date: 2018-11-11
toc: true
linktitle: Lambda functions
title: Integration with AWS Lambda functions
since: 1.0
source: https://github.com/devopsfaith/krakend-lambda
weight: 110
notoc: true
menu:
  documentation:
    parent: backends
---

TThe Lambda integration allows you to invoke Amazon Lambda functions on a KrakenD endpoint call.

Add lambda invocations into KrakenD as any other regular backend. The inclusion requires you to add the code in the `extra_config` of your `backend` section, using the `github.com/devopsfaith/krakend-lambda` namespace.

The KrakenD machine needs to have the AWS credentials in the default file, `~/.aws/credentials`.

The supported parameters are:

*   `function_name`: Name of the lambda function as saved in the AWS service.
*   `function_param_name`: The endpoint `{placeholder}` that sets the function name. You have to choose between `function_name` and `function_param_name` but not both.
*   `region`: The AWS identifier region (e.g.: `us-east-1`, `eu-west-2`, etc. )
*   `max_retries`: Maximum times you want to execute the function until you have a successful response.
*   `endpoint`: An optional parameter to customize the Lambda endpoint to call. Useful when Localstack is used for testing.

### Example: call a lambda

When the KrakenD endpoint is attached to the same Lambda, use this configuration:

    "github.com/devopsfaith/krakend-lambda": {
        "function_name": "lambda-function",
        "region": "us-west1",
        "max_retries": 1
    }

### Example: Call a lambda dynamically

When the name of the Lambda to depends on a parameter passed in the endpoint, use this configuration instead:

    "github.com/devopsfaith/krakend-lambda": {
        "function_param_name": "function",
        "region": "us-west1",
        "max_retries": 1
    }

In this example, the `function_param_name` is telling us that there is a placeholder `{function}` in the endpoint. For instance: `"enpoint": "/foo/{function}"`. The placeholder `{function}` resolves the name of the Lambda.

Following the code, a call `GET /foo/my-lambda` would produce a call to the `my-lambda` function in AWS.
