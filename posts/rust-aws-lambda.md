---
title: Running Rust code in the cloud with AWS Lambda
date: 2024-02-14
tags:
  - rust
  - software
  - aws
  - cloud
layout: layouts/post.njk
---

## Introduction

Running Rust code in AWS lambda has many benefits, I personally enjoy working with Rust, the strict compiler and the cargo ecosystem. Having a quick way to create serverless functions with Rust seems like a great idea.

We'll go through the steps necessary to create a lambda using cloudformation syntax, the AWS CLI and preparing our Rust code to be deployed.

## Required tools

- rustup: for adding the new compilation target (needed for Mac)
- [cargo lambda](https://www.cargo-lambda.info/): Helps building rust packages that will target AWS Lambda runtime
- AWS CLI: For deploying the cloudformation stack

## Writing our Rust program

The code we'll use is taken from the examples of the [lambda_http](https://github.com/awslabs/aws-lambda-rust-runtime/tree/main/examples) repository. It's a very simple function that returns a string, it's enough to test if things work and a useful starter template:

```rust
/// Extracted from  https://github.com/awslabs/aws-lambda-rust-runtime/tree/main/examples

use lambda_http::{run, service_fn, Body, Error, Request, Response};
use tracing_subscriber;
use tracing;
async fn function_handler(_event: Request) -> Result<Response<Body>, Error> {
    // Extract some useful information from the request

    // Return something that implements IntoResponse.
    // It will be serialized to the right response event automatically by the runtime
    let resp = Response::builder()
        .status(200)
        .header("content-type", "text/html")
        .body("Hello AWS Lambda HTTP request".into())
        .map_err(Box::new)?;
    Ok(resp)
}

#[tokio::main]
async fn main() -> Result<(), Error> {
    // required to enable CloudWatch error logging by the runtime
    tracing_subscriber::fmt()
        .with_max_level(tracing::Level::INFO)
        // disable printing the name of the module in every log line.
        .with_target(false)
        // disabling time is handy because CloudWatch will add the ingestion time.
        .without_time()
        .init();

    run(service_fn(function_handler)).await
}
```
The idea is simple, you wrap your function handler in a `service_fn` and it's itself wrapped in a `run` function. Tokyo is used to provide the async capabilities.

## lambda_runtime vs lambda_http

When I tried the first time I used an example that used `lambda_runtime` and got some issues because lambda_runtime does not understand AWS Api Gateway context. I found out that `lambda_http` is the right choice for this case as it's a higher level library that abstracts the API Gateway request/response objects.

## Compiling the Rust code

I'm using a Mac, the first time I tried to deploy the package I got an error due to executable incompatibilities. I learned I had to target a linux platform so it's compatible with the AWS Lambda environment.

Installing the linux target with rustup:

```bash
rustup target add x86_64-unknown-linux-musl
```

And now building the package for the new target using cargo lambda:

```bash
cargo lambda build --target=x86_64-unknown-linux-musl --release
```

## Creating the lambda zip package.

We now have to create a zip that contains the compiled artifacts created in the compilation step.

The `cargo lambda` tool creates a file called `bootstrap`in the `target/lambda/your-project-name` directory and this is the only file that we need to zip.

Run the following command to create the zip (from the directory that contains the bootstrap file):

```bash
zip -r lambda-package.zip 
```


## Creating the S3 bucket to store the lambda package:

Let's create an encrypted S3 bucket to store our lambda zip package.

```bash
# Replace `rust-lambda-code-test` with your bucket name!
aws s3api create-bucket --bucket rust-lambda-code-test --region us-east-1
aws s3api put-bucket-encryption --bucket rust-lambda-code-test --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

## The Cloudformation template 

I used this cloudformation template to create the Lambda in AWS, make sure to adjust the path of the .zip package we created before:

The parameters defined by this template are:

1. BucketName: The name of the S3 bucket where the Lambda deployment package is located
2. PathToLambdaZip: Local path to the zip package created before

** Note: I put the cloudformation template in a file called in the path: cloud-formation/rust-lambda.yaml inside my Rust project, this path will be needed create the stack later **


```yaml
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  BucketName:
    Type: String
    Description: The name of the S3 bucket where the Lambda deployment package is located
    Default: your-bucket-name
  PathToLambdaZip:
    Type: String
    Description: The path to the Lambda deployment package in the S3 bucket
    Default: path/to/lambda.zip
Resources:
  RustLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Runtime: provided.al2
      Code:
        S3Bucket: !Ref BucketName
        S3Key: !Ref PathToLambdaZip
      MemorySize: 128
      Timeout: 15

  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: 'sts:AssumeRole'
      Policies:
        - PolicyName: LambdaExecutionPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action: 'logs:*'
                Resource: 'arn:aws:logs:*:*:*'

  MyApi:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: RustServiceAPI

  Resource:
    Type: AWS::ApiGateway::Resource
    DependsOn: MyApi
    Properties:
      RestApiId: !Ref MyApi
      ParentId: !GetAtt 'MyApi.RootResourceId'
      PathPart: rustservice

  Method:
    Type: AWS::ApiGateway::Method
    DependsOn: Resource
    Properties:
      RestApiId: !Ref MyApi
      ResourceId: !Ref Resource
      HttpMethod: POST
      AuthorizationType: NONE
      Integration:
        Type: AWS_PROXY
        IntegrationHttpMethod: POST
        Uri: !Sub 'arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${RustLambdaFunction.Arn}/invocations'
      MethodResponses: []

  LambdaInvokePermission:
    Type: AWS::Lambda::Permission
    Properties:
      FunctionName: !Ref RustLambdaFunction
      Action: lambda:InvokeFunction
      Principal: apigateway.amazonaws.com
      SourceArn: !Sub 'arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${MyApi}/*/*/*'
  ApiDeployment:
    Type: AWS::ApiGateway::Deployment
    DependsOn: Method
    Properties:
      RestApiId: !Ref MyApi
      StageName: Dev
Outputs:
  ApiEndpoint:
    Description: "API Gateway endpoint URL for the Rust service"
    Value: !Sub "https://${MyApi}.execute-api.${AWS::Region}.amazonaws.com/Dev/rustservice"
```

## Deploying the Cloudformation stack

To deploy de stack, run the following command providing the required parameters for bucket name and zip package location. The AWS CLI tool will start the process of deploying and you can see the result on the AWS Console:

```bash
aws cloudformation create-stack \
  --stack-name RustLambdaServiceTestStack \
  --template-body file://cloud-formation/rust-lambda.yaml \
  --parameters ParameterKey=BucketName,ParameterValue=rust-lambda-code-test \
               ParameterKey=PathToLambdaZip,ParameterValue=lambda-package.zip \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```

## Updating our Rust code

If you change your code, you'll need to build and create the zip file again. Then run the following command to update the stack:

```bash
aws cloudformation update-stack \
  --stack-name RustLambdaServiceTestStack \
  --template-body file://cloud-formation/rust-lambda.yaml \
  --parameters ParameterKey=BucketName,ParameterValue=rust-lambda-code-test \
               ParameterKey=PathToLambdaZip,ParameterValue=lambda-package-v2.zip \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
```
## Obtaining the API URL

Go to the AWS Console -> Cloud Formation and look for the stack you created. Click on the Outputs tab and you'll find the URL of the API endpoint you can use.

## Testing the Lambda

To test that our lambda works send an HTTP request using curl or any other tool:

```bash
curl -X POST -H "Content-Type: application/json" https://<generated-id>.execute-api.us-east-1.amazonaws.com/Dev/rustservice
```

Result:

`Hello AWS Lambda HTTP request`

## Conclusion

Having a way to quickly deploy Rust code as serverless functions is very useful. Not being dependent on a runtime makes things simpler in terms of compiling/updating and deployment of our code.

