---
lastmod: 2022-06-09
old_version: true
date: 2018-11-11
toc: true
linktitle: Lambda functions
title: Integration with AWS Lambda functions
weight: 110
images:
    - /images/krakend-lambda.png
menu:
  community_v2.1:
    parent: "050 Backends Configuration"
notoc: true
meta:
  since: 1.0
  source: https://github.com/krakendio/krakend-lambda
  namespace:
  - backend/lambda
  scope:
  - backend
---

The Lambda integration allows you to **invoke Amazon Lambda functions** on a KrakenD endpoint call. The content returned by the lambda function can be treated and manipulated as any other backend.

The **payload** that is sent to the Lambda function comes from the request and depends on the method used by the `endpoint`:

*   Method `GET`: The payload contains all the request parameters.
*   Non-`GET` methods: The payload is defined by the content of the **body** in the request.

You don't need to set an Amazon API Gateway in the middle as KrakenD does this job for you.


## Lambda configuration

{{< note title="Dummy hosts and url_pattern" type="info" >}}
Notice in the examples that the `host` and `url_pattern` are needed as per the [backend definition](/docs/v2.1/backends/), but KrakenD will never use them. Feel free to add any value in there, but they must be present.
{{< /note >}}

The inclusion requires you to add the code in the `extra_config` of your `backend` section, using the `backend/lambda` namespace.

The supported parameters are:

*   `function_name`: Name of the lambda function as saved in the AWS service.
*   `function_param_name`: The endpoint `{placeholder}` that sets the function name. You have to choose between `function_name` and `function_param_name` but not both.
*   `region`: The AWS identifier region (e.g.: `us-east-1`, `eu-west-2`, etc. )
*   `max_retries`: Maximum times you want to execute the function until you have a successful response.
*   `endpoint`: An optional parameter to customize the Lambda endpoint to call. Useful when Localstack is used for testing.

{{< note title="Parameters' first character uppercased" >}}
Notice the capitalization of the first letter of the parameter names at the configuration in the examples below. For instance, when an endpoint parameter is defined as `{route}`, define it in the config as `Route`.
{{< /note >}}


{{< schema version="v2.1" data="backend/lambda.json" >}}

### Authentication and connectivity

The KrakenD machine needs to have connectivity with your AWS account and have the credentials to do so. There are several ways you can achieve this:

- By copying your AWS credentials in the default file, `~/.aws/credentials` (and maybe an additional `~/.aws/config` and the env var `AWS_PROFILE` if you have several profiles)
- By passing the environment variables with at least `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` (and maybe `AWS_REGION`) when starting KrakenD.
- By having an IAM user with a policy and execution role that let you invoke the function from the machine

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
    -v "$PWD:/etc/krakend" {{< product image >}}:v2.1
{{< /terminal >}}

Passing the credentials directly:

{{< terminal title="Term" >}}
docker run --rm -it -p "8080:8080" \
    -e "AWS_ACCESS_KEY_ID=XXX" \
    -e "AWS_SECRET_ACCESS_KEY=XXX" \
    -e "AWS_REGION=eu-west-1" \
    -v "$PWD:/etc/krakend" {{< product image >}}:v2.1
{{< /terminal >}}


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
        statusCode: 200,
        body: {"message": "Hello World"},

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
