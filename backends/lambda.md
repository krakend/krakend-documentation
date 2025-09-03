---
lastmod: 2023-06-20
date: 2018-11-11
linktitle: Lambda functions
title: AWS Lambda Integration
description: Integrate AWS Lambda functions into KrakenD API Gateway to leverage serverless computing for scalable and event-driven API processing
weight: 150
images:
    - /images/krakend-lambda.png
dark_header_image: true
menu:
  community_current:
    parent: "050 Non-REST Connectivity"
notoc: false
meta:
  since: v1.0
  source: https://github.com/krakend/krakend-lambda
  namespace:
  - backend/lambda
  scope:
  - backend
---

The Lambda integration allows you to **invoke Amazon Lambda functions** on a KrakenD endpoint call. The content returned by the lambda function can be treated and manipulated as any other backend.

The **payload** that is sent to the Lambda function comes from the request and depends on the method used by the `endpoint`:

*   Method `GET`: The payload contains all the request parameters.
*   Non-`GET` methods: The payload is defined by the content of the **body** in the request.

You don't need to set an Amazon API Gateway in the middle, as KrakenD does this job for you.


## Lambda configuration

{{< note title="Dummy hosts and url_pattern" type="info" >}}
Notice in the examples that the `host` and `url_pattern` are needed as per the [backend definition](/docs/backends/), but KrakenD will never use them when connecting to a Lambda. Feel free to add any value in there, but the entry must be present.
{{< /note >}}

The inclusion requires you to add the code in the `extra_config` of your `backend` section using the `backend/lambda` namespace.

The supported parameters are:




{{< schema data="backend/lambda.json" >}}

{{< note title="Parameters' first character uppercased" >}}
Notice the capitalization of the first letter of the parameter names at the configuration in the examples below. For instance, when an endpoint parameter is defined as `{route}`, define it in the config as `Route`.
{{< /note >}}

When passing the lambda function name, you can use any of the following formats:

- Name only: `my-function`
- Name and version: `my-function:v1`
- Function ARN: `arn:aws:lambda:us-west-2:123456789012:function:my-function`
- Partial ARN: `123456789012:function:my-function`

### Authentication and connectivity

The KrakenD machine needs connectivity with your AWS account and the credentials to do so. There are several ways you can achieve this:

- Copying your AWS credentials in the default file, `~/.aws/credentials` (and maybe an additional `~/.aws/config` and the env var `AWS_PROFILE` if you have several profiles)
- Passing the environment variables with at least `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` (and maybe `AWS_REGION`) when starting KrakenD.
- Having an IAM user with a policy and execution role that lets you invoke the function from the machine

When setting the credentials, ensure the Lambda is callable within the KrakenD box with the selected method.

If your machine has the AWS CLI installed, you can test your Lambda:

{{< terminal title="Invoking a Lambda function" >}}
aws lambda invoke --region us-east-1 --function-name myLambdaFunction --output json result.json
{{< /terminal >}}

#### Authentication examples
Mounting an existing `.aws` directory with the credentials in it (notice that the home of the Docker user is `krakend`):

{{< terminal title="Term" >}}
docker run --rm -it -p "8080:8080" \
    -e "AWS_PROFILE=default" \
    -v "/home/user/.aws:/home/krakend/.aws:ro" \
    -v "$PWD:/etc/krakend" {{< product image >}}:{{< product latest_version >}}
{{< /terminal >}}

Passing the credentials directly:

{{< terminal title="Term" >}}
docker run --rm -it -p "8080:8080" \
    -e "AWS_ACCESS_KEY_ID=XXX" \
    -e "AWS_SECRET_ACCESS_KEY=XXX" \
    -e "AWS_REGION=eu-west-1" \
    -v "$PWD:/etc/krakend" {{< product image >}}:{{< product latest_version >}}
{{< /terminal >}}

### Header forwarding
**The official library of AWS has a limitation preventing KrakenD from sending any headers**, yet it is impossible to forward them even when you add them under `input_headers`.

If you require such functionality, you should create a custom payload ([like this one](https://github.com/awsdocs/aws-lambda-developer-guide/blob/main/sample-apps/nodejs-apig/event.json)) with additional data containing the headers (and anything else than the body itself).

### Canary testing of Lambda functions
You can easily do **Canary** or **A/B testing** with Lambda functions in combination with a Lua script like the one provided below.

For example, the following configuration takes the function name from a parameter that is not defined anywhere. Instead, a Lua script will set it as per our AB test or canary rules.

The configuration would be:

```json
{
  "endpoint": "/call-a-lambda",
  "backend": [
    {
      "host": ["ignore"],
      "url_pattern": "/ignore",
      "extra_config": {
        "backend/lambda": {
          "function_param_name": "Function_name",
          "region": "eu-west-1",
          "max_retries": 1
        },
        "modifier/lua-backend": {
            "sources": ["canary.lua"],
            "pre": "canaryLambda(request.load())",
            "allow_open_libs": true
        }
      }
    }
  ]
}
```
And the content of the `canary.lua` file would be:

```lua
function canaryLambda( req )
  local rand = math.random(0, 100)
  if rand < 20
  then
    req:params("Function_name", "my-function:3")
  else
    req:params("Function_name", "my-function:2")
  end
end
```

As you can read from the code, 20% of the requests will invoke `my-function:3`, while the 80% remaining will fall into the older `my-function:2`.

For more Canary Release examples (not only Lambda) see this [blog post](/blog/canary-releases/)

## Example: Associate a lambda to a backend

When you associate a KrakenD endpoint to a unique lambda function, use this configuration:

```json
{
  "endpoint": "/call-a-lambda",
  "backend": [
    {
      "host": ["ignore"],
      "url_pattern": "/ignore",
      "extra_config": {
        "backend/lambda": {
          "function_name": "myLambdaFunction",
          "region": "us-east-1",
          "max_retries": 1
        }
      }
    }
  ]
}
```

Here is an example of Lambda code that could run a Hello World (NodeJS):

```js
exports.handler = async (event) => {
    // TODO implement
    const response = {
        "message": "Hello World",
    };
    return response;
};
```

## Example: Take the lambda from the URL

When the name of the Lambda depends on a parameter passed in the endpoint, use this configuration instead:

```json
{
  "endpoint": "/call-a-lambda/{lambda}",
  "backend": [
    {
      "host": ["ignore"],
      "url_pattern": "/ignore",
      "extra_config": {
        "backend/lambda": {
          "function_param_name": "Lambda",
          "region": "us-west1",
          "max_retries": 1
        }
      }
    }
  ]
}
```

In this example, the `function_param_name` tells us which is the placeholder in the endpoint setting the Lambda. In this case, `{lambda}`.

Following the code, a call `GET /call-a-lambda/my-lambda` would produce a call to the `My-lambda` function in AWS.
