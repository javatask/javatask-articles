---
title: "Deploying a Single Page Application (SPA) on AWS: A Beginner's Guide. Part 3. REST API"
seoTitle: "SPA Deployment AWS: Beginner's Pt.3, REST API"
seoDescription: "Master AWS deployment using a beginner's guide to secure REST API for SPAs, covering API Gateway, private container registry, and EC2 setup"
datePublished: Sun Nov 05 2023 16:00:12 GMT+0000 (Coordinated Universal Time)
cuid: clolntcm9000409l5ditbfple
slug: deploying-a-single-page-application-spa-on-aws-a-beginners-guide-part-3-rest-api
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697629150815/a7cb2814-a4d9-4a81-bb53-f3a6fc7fdd0b.png
tags: docker, aws, backend, rest-api, containers

---

## Introduction

In this section, we will construct a secure REST API for the backend of our application. This involves setting up a private container registry, configuring a Linux Virtual Machine (AWS EC2 instance), and utilizing API Gateway to proxy requests to our VM with the REST API server.

![Resources created for the Backend](https://cdn.hashnode.com/res/hashnode/image/upload/v1690551481770/b0ed792b-2134-40c6-bd60-5cf8b8ded00f.png align="left")

*Figure 1: Resources created for the Backend*

# Source code repository

[javatask/aws-spa-beginners-guide: Code for the blog series on porting single page application to AWS (](https://github.com/javatask/aws-spa-beginners-guide)[github.com](http://github.com)[)](https://github.com/javatask/aws-spa-beginners-guide)

## API Gateway

You might wonder, why we need API Gateway when our REST API server can handle requests on its own. The answer lies in the API gateway pattern and the benefits it offers.

### API Gateway Pattern

The API gateway pattern is an architectural approach that simplifies and optimizes communication between clients and backend services. It acts as a single entry point for clients to access these services, abstracting away implementation details and providing additional functionalities such as authentication, authorization, load balancing, caching, logging, and more.

This pattern resembles the facade pattern from object-oriented design but is tailored for distributed systems, employing reverse proxies or gateway routing and a synchronous communication model. The API gateway functions as a reverse proxy, routing requests to internal backend endpoints based on predefined rules. It also handles request and response translation between clients and backends, managing various types, formats, and protocols.

The benefits of the API gateway pattern include:

* **Simplified Client-Side Development:** It offers a uniform and consistent interface for clients, regardless of the number, location, or technology of the backends.
    
* **Reduced Latency and Bandwidth Consumption:** By aggregating and transforming data from multiple backends into a single response, it minimizes network latency and bandwidth usage.
    
* **Enhanced Security and Reliability:** Common features like authentication, authorization, encryption, rate limiting, circuit breaking, and retrying are implemented at the gateway level, enhancing application security and reliability.
    
* **Scalability and Flexibility:** Dynamic routing and backend discovery allow for easy scaling and the addition or removal of backends without impacting clients.
    
* **Monitoring and Logging:** It facilitates application monitoring and logging by collecting and analyzing metrics and traces from the gateway and backends.
    

In summary, the API gateway pattern is a valuable design pattern for applications with multiple backends. It provides a single entry point, streamlines client-side development, reduces latency and bandwidth usage, bolsters security and reliability, enables scalability and flexibility, and simplifies monitoring and logging.

API Gateway is crucial for decoupling Backend hosting options from Frontend. It allows us to make significant changes to the Backend hosting without affecting Frontend implementation.

Now that we understand the importance of API Gateway, let's set up a private container registry to proceed with our CloudFormation (CFN) template.

## Private Container Registry

To streamline the deployment of our "complex" backend, we will employ containerization. To maintain security and the ability to use custom enterprise containers, we'll utilize a private container registry.

To create a private Elastic Container Registry (ECR), execute the following command:

```bash
aws ecr create-repository --repository-name rest-api
```

I recommend checking the AWS ECR UI/Console for instructions on logging in, building, and pushing your custom container. As of 2023, these are the commands used:

1. Retrieve an authentication token and authenticate your Docker client to your registry:
    
    ```bash
    aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 1111111111.dkr.ecr.eu-central-1.amazonaws.com
    ```
    
2. Build your Docker image. You can refer to the [instructions here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html) if you need to build a Docker file from scratch. You can skip this step if your image is already built:
    
    ```bash
    docker build -t test .
    ```
    
3. After the build is complete, tag your image to push it to your newly created AWS repository:
    
    ```bash
    docker tag test:latest 11111111.dkr.ecr.eu-central-1.amazonaws.com/test:latest
    ```
    
4. Push the image to your AWS repository:
    
    ```bash
    docker push 11111111111.dkr.ecr.eu-central-1.amazonaws.com/test:latest
    ```
    

You will need the container URI `11111111111.dkr.ecr.eu-central-1.amazonaws.com/test:latest` to deploy the Backend.

Now, let's proceed with deploying the Backend.

## Deploy the Backend

For the deployment, we will use a CloudFormation (CFN) template file called `api-backend.yaml`. You need to set the value of the `ContainerURI` parameter to point to your container URI.

> **Note:** You might need to change the `AWS::EC2::Instance` `InstanceType` and/or `ImageId` if you deploy this template in another region or with a different architecture. I have tested this template in the `eu-central-1` region.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Creating EC2 instance for API Backend

Parameters:
  ContainerURI:
    Type: String
    Default: 11111111.dkr.ecr.eu-central-1.amazonaws.com/rest-api:latest
    Description: Container URI to launch

Resources:
  # Api Gateway
  ApiGatewayRestApi:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: !Sub ${AWS::StackName}-backend-proxy
  ApiGatewayResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      ParentId:
        !GetAtt ApiGatewayRestApi.RootResourceId
      PathPart: '{proxy+}'
      RestApiId:
        !Ref ApiGatewayRestApi
  ApiGatewayMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      HttpMethod: ANY
      ResourceId:
        !Ref ApiGatewayResource
      RestApiId:
        !Ref ApiGatewayRestApi
      AuthorizationType: NONE
      RequestParameters:
        method.request.path.proxy: true
      Integration:
        RequestParameters:
          integration.request.path.proxy: method.request.path.proxy
        Type: HTTP_PROXY
        IntegrationHttpMethod: ANY
        Uri: !Sub "http://${EC2Instance.PublicDnsName}:8080/{proxy}"
  ApiGatewayDeployment:
    Type: 'AWS::ApiGateway::Deployment'
    DependsOn:
      - ApiGatewayMethod
    Properties:
      RestApiId:
        !Ref ApiGatewayRestApi
      StageName: production

# VM backend
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0e00e602389e469a3
      IamInstanceProfile: !Ref EC2InstanceProfile
      SecurityGroups:
        - !Ref EC2SecurityGroup
      Tags:
        - Key: Name
          Value: CloudFormation-EC2-Instance
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          sudo dnf -y update
          sudo dnf -y install docker
          sudo systemctl start docker
          sudo systemctl enable docker
          aws ecr get-login-password --region ${AWS::Region} | sudo docker login --username AWS --password-stdin ${AWS::AccountId}.dkr.ecr.${AWS::Region}.amazonaws.com
          sudo docker pull ${ContainerURI}
          sudo docker run -d -p 8080:8080 ${ContainerURI}

  EC2Role:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - ec2.amazonaws.com
            Action:
              - 'sts:AssumeRole'
      Path: /
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

  EC2InstanceProfile: 
    Type: "AWS::IAM::InstanceProfile"
    Properties: 
      Roles: 
        - !Ref EC2Role

  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow inbound HTTP traffic on port 8080
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 8080
          ToPort: 8080
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceId:
    Value: !Ref EC2Instance
  ApiGatewayURL:
    Value: !Sub "https://${ApiGatewayRestApi}.execute-api.${AWS::Region}.amazonaws.com/production"
```

This AWS CloudFormation template orchestrates several components to deploy an API backend using an Amazon EC2 instance, which runs Docker to pull and execute a specific container, exposing the API through an Amazon API Gateway.

**EC2 Instance Configuration:**

* Defines a Docker image (`ContainerURI`) to be pulled from an Amazon ECR repository, which represents the backend REST API application.
    
* Creates an EC2 instance with type `t2.micro` using an AMI image specified by the `ImageId` parameter.
    
* Associate the EC2 instance with an IAM instance profile (`EC2InstanceProfile`), allowing the instance to pull Docker images from ECR.
    
* Assign the EC2 instance to a security group (`EC2SecurityGroup`) that allows inbound HTTP traffic on port 8080.
    
* Executes a startup script (`UserData`) that updates the instance, installs Docker, starts Docker, logs into ECR, pulls the Docker image, and runs the Docker container, exposing the application on port 8080.
    

**IAM Role and Instance Profile Configuration:**

* Creates an IAM role (`EC2Role`) for the EC2 service, allowing it to assume this role. The role is the manage policy `AmazonEC2ContainerRegistryReadOnly` attached, granting read-only access to Amazon ECR.
    
* Associate the EC2 instance with this IAM role by creating an IAM instance profile (`EC2InstanceProfile`) and linking the IAM role.
    

**Security Group Configuration:**

* Establishes a security group (`EC2SecurityGroup`) for the EC2 instance, allowing inbound traffic for TCP on port 8080 from any IP address (0.0.0.0/0).
    

**API Gateway Configuration:**

* Creates an API Gateway (`ApiGatewayRestApi`), a resource (`ApiGatewayResource`), and an HTTP method (`ApiGatewayMethod`) that accepts any HTTP verb and doesn't require authentication (AuthorizationType: NONE).
    
* The integration type is HTTP\_PROXY, setting up the API Gateway as a proxy to the EC2 instance. The `Uri` property dynamically references the public DNS name of the EC2 instance, with a path parameter `{proxy}`, allowing it to pass requests to the backend service running in the Docker container on the EC2 instance.
    
* Deploys the API Gateway (`ApiGatewayDeployment`) to a stage named 'production'.
    

**Outputs:**

* The template provides two outputs: the instance ID of the EC2 instance and the URL of the API Gateway in the 'production' stage.
    

This setup is beneficial for creating a publicly accessible REST API served by a containerized application running on an EC2 instance.

To deploy the stack, navigate to the `repository` folder `p3` and run:

```bash
aws cloudformation deploy \
    --stack-name one-click-backend \
    --template-file ./templates/backend.yaml \
    --parameter-overrides ContainerURI=11111111.dkr.ecr.eu-central-1.amazonaws.com/xxx:spring-rest-api \
    --capabilities CAPABILITY_IAM
```

Now, let's test our newly deployed Backend:

```bash
curl https://xxxxxx.execute-api.eu-central-1.amazonaws.com/production/api/greeting
```

You should receive the following JSON response:

```json
{"id":1,"content":"Hello, World!"}
```

## Drawbacks of EC2 as a Backend

This section will highlight the main drawbacks of this configuration and potential mitigation strategies.

One drawback is that both my EC2 Instance and API Gateway URL are publicly reachable. To address this, you can consider disabling the Public IP, using Api Gateway VPC Link, adding an authorizer to Api Gateway, and placing it behind AWS CloudFront.

Another challenge is the workflow for pushing new versions of your backend implementation. Here, I refer to practices like [blue-green deployments](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/bluegreen-deployments.html) or [rolling deployments](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/rolling-deployments.html). There's no straightforward way to establish a CI/CD pipeline apart from concealing EC2 behind the Load Balancer and building new AMI images with the latest version of your Backend container.

I have shown you this approach as a more developer-focused method that allows for easy debugging of various components of your application.

## Summary

In this article, we've discussed how to build a secure REST API for the backend by setting up a private container registry, configuring a Linux Virtual Machine (AWS EC2 instance), and using API Gateway to proxy requests to the REST API server. We've explained the benefits of the API gateway pattern, such as simplifying client-side development, enhancing security and reliability, enabling scalability, and simplifying monitoring and logging. We've provided a step-by-step guide on creating a private Elastic Container Registry (ECR), deploying the backend using a CloudFormation template, and testing the deployed backend. Finally, we've discussed the drawbacks of this configuration and potential improvements.

In the next part, we will add an authentication layer with AWS Cognito to our Backend and establish the necessary connections between the Backend and Frontend to create an end-to-end (E2E) experience.