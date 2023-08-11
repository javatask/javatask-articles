---
title: "Migrating your classic Single Page Application to AWS. Part 8"
slug: migrating-your-classic-single-page-application-to-aws-part-8
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1691770389576/1ae076ff-8623-4414-9173-1419801ef55e.png

---

# Introduction

Serverless architecture and AWS Lambda have revolutionized the way developers build and deploy applications. By eliminating the need to manage servers, developers can focus more on writing code and delivering features. This article will explore the benefits of serverless, such as reduced operational overhead, cost-effectiveness, and scalability. Additionally, we will discuss AWS Lambda, a popular serverless computing platform, and how it can help streamline your application development process.

A great resource to understand the theory behind serverless is [serverlessland.com](http://serverlessland.com)

# Tooling

[AWS SAM (Serverless Application Model)](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-getting-started.html) is a framework that simplifies the development and deployment of serverless applications on AWS. It extends AWS CloudFormation to provide a simplified way of defining the resources and functions required for your application. To install AWS SAM, you can use the AWS SAM CLI, which can be installed via package managers like Homebrew, pip, or by downloading the installer from the official AWS SAM GitHub repository.

# Java Code + SAM Cloudformation template = Production Backend

# What is the price of Serverless?

Refactoring your code to make it serverless and compatible with AWS Lambda is essential to leverage the benefits of serverless architecture. This process involves adapting your application's structure and codebase to work with Lambda functions, which are stateless, event-driven, and automatically managed by AWS. By doing so, you can take advantage of reduced operational overhead, cost-effectiveness, and improved scalability without the need to manage servers.

## How complex refactoring is?

With Spring Boot 3 Application, it's as simple as importing [one library from AWS](https://github.com/awslabs/aws-serverless-java-container) to wrap your Spring Boot App into an AWS Lambda handler:

```java
public class GreetingLambdaHandler implements RequestStreamHandler {
    private static SpringBootLambdaContainerHandler<HttpApiV2ProxyRequest, AwsProxyResponse> handler;
    static {
        try {
            handler = SpringBootLambdaContainerHandler.getHttpApiV2ProxyHandler(RestServiceApplication.class);
        } catch (ContainerInitializationException e) {
            e.printStackTrace();
            throw new RuntimeException("Could not initialize Spring Boot application", e);
        }
    }

    @Override
    public void handleRequest(InputStream inputStream, OutputStream outputStream, Context context)
            throws IOException {
        handler.proxyStream(inputStream, outputStream, context);
    }
}
```

That's all!

## How to deploy the App?

Here CFN template for our Spring Rest API backend:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Example Rest Api Serverless for Spring Boot 3

Resources:
  MyRestApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: com.example.restservice.GreetingLambdaHandler::handleRequest
      Runtime: java17
      CodeUri: .
      MemorySize: 512
      Policies: AWSLambdaBasicExecutionRole
      Timeout: 60
      Events:
        HttpApiEvent:
          Type: HttpApi
          Properties:
            TimeoutInMillis: 20000
            PayloadFormatVersion: "2.0"

Outputs:
  RestApi:
    Description: URL for application
    Value: !Sub "https://${ServerlessHttpApi}.execute-api.${AWS::Region}.amazonaws.com/api"
```

Let's build and deploy it:

```bash
sam build
sam deploy -g
```

Use URL:

```bash
curl https://${ServerlessHttpApi}.execute-api.${AWS::Region}.amazonaws.com/api/greeting
```

So simple ;)

# Let's deep dive into CFN template

The provided CloudFormation template is a description of AWS resources to be created or configured, focusing on a serverless architecture for hosting a REST API using Spring Boot 3 and Java 17.

Here's a breakdown of each part:

1. **AWSTemplateFormatVersion**: Specifies the version of the CloudFormation template, in this case, '2010-09-09'.
    
2. **Transform**: Indicates that the template uses the AWS Serverless Application Model (SAM) transformation named 'AWS::Serverless-2016-10-31'. This transformation provides a simplified syntax for describing serverless resources such as Lambda functions and APIs.
    
3. **Description**: Provides a brief description of the template's purpose, specifically mentioning that it's an example REST API using serverless technology for Spring Boot 3.
    
4. **Resources**: The main section, where all the resources to be created or configured are defined:
    
    * **MyRestApiFunction**: Describes a Lambda function that will handle the REST API requests.
        
        * **Handler**: Specifies the Java class and method that AWS Lambda will call to start executing your function.
            
        * **Runtime**: Specifies the runtime environment (Java 17).
            
        * **CodeUri**: The location of the code for the Lambda function (in this case, the current directory).
            
        * **MemorySize**: The amount of memory available to the function during execution (512 MB).
            
        * **Policies**: The permissions assigned to the function, here using a predefined basic execution role.
            
        * **Timeout**: The maximum amount of time the function is allowed to run (60 seconds).
            
        * **Events**: Defines an HTTP API event that triggers the Lambda function. It sets a specific timeout and payload format version for the event.
            
5. **Outputs**: The section defines values that you can import into other stacks, return in response, or view in the AWS CloudFormation console.
    
    * **RestApi**: The output provides the URL for the REST API application, constructed dynamically using the region where the stack is launched.
        

Overall, this CloudFormation template represents a serverless application for a RESTful API using Spring Boot 3, capable of being deployed on AWS using Lambda and other serverless technologies. It includes configurations and properties that define how the API and Lambda functions should behave.

# Summary

It feels like the shortest article in the Series :)

In this article, we discuss the benefits of serverless architecture and AWS Lambda for application development, including reduced operational overhead, cost-effectiveness, and scalability. We cover the AWS SAM framework, the process of refactoring code for serverless compatibility, and provide an example of deploying a Spring Boot 3 REST API using a CloudFormation template.