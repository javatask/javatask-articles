---
title: "Deploying a Single Page Application (SPA) on AWS: A Beginner's Guide. Part 5. Connecting dots"
seoTitle: "SPA Deployment on AWS: Beginner's Guide Pt.5"
seoDescription: "Beginner's guide to securely connecting SPA components using AWS CloudFormation"
datePublished: Wed Oct 18 2023 14:38:42 GMT+0000 (Coordinated Universal Time)
cuid: clnvuz7p3000e09js6n8w2jdy
slug: deploying-a-single-page-application-spa-on-aws-a-beginners-guide-part-5-connecting-dots
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697631364898/63715578-9838-48f7-8d90-59160301b265.png
tags: cloudformation, aws, spa, full-stack, iac

---

## Introduction

In this section, we will connect the authentication, backend, and frontend components to create a secure Single Page Application (SPA) using a CloudFormation (CFN) template. While the template isn't perfect, it's close to being production-ready. Figure 1 illustrates how all the parts come together.

![All the parts connected](https://cdn.hashnode.com/res/hashnode/image/upload/v1690993456357/91c798b7-820a-4eaa-bb50-812b4fe8961e.png align="left")

*Figure 1: All the parts connected*

Let's briefly review what we've covered in the previous parts and the changes needed to integrate them:

* **Part 1**: Theory. No changes
    
* **Part 2**: Frontend setup, including the deployment of an AWS CloudFront CDN Distribution. In this part, we'll update its CFN template to route frontend `/api` queries to the backend.
    
* **Part 3**: Backend setup, configuring a VM (EC2 Instance) to run the REST API Server and an API Gateway to proxy requests to the VM. In this part, we'll also make changes to protect the API Gateway using our OpenID Server JSON Web Token (JWT). We'll also change the backend endpoint from `/greeting` to `/api/greeting`.
    
* **Part 4**: Identity Provider, where we deployed AWS Cognito as the OpenID Server for user authentication. No changes are needed in this part for the initial CFN template.
    

# Source code repository

[javatask/aws-spa-beginners-guide: Code for the blog series on porting single page application to AWS (](https://github.com/javatask/aws-spa-beginners-guide)[github.com](http://github.com)[)](https://github.com/javatask/aws-spa-beginners-guide)

## Connect Frontend to the Backend

To connect the frontend and backend, we'll make changes to the `frontend.yml` file. You can find the entire CFN template in the repository.

```yaml
AssetsDistribution:
    Type: AWS::CloudFront::Distribution
      Properties:
      DistributionConfig:
       ...
        - Id: APIBackend
          DomainName: !Ref APIEndpoint
          OriginPath: !Sub "/${APIEndpointStage}"
          CustomOriginConfig:
            OriginProtocolPolicy: https-only
       ...
       CacheBehaviors:
          - PathPattern: "/api/*"
            AllowedMethods:
            - GET
            - POST
            - PATCH
            - PUT
            - DELETE
            - HEAD
            - OPTIONS
            TargetOriginId: APIBackend
            CachePolicyId: 4135ea2d-6df8-44a3-9df3-4b5a84be39ad # CachingDisabled
            OriginRequestPolicyId: b689b0a8-53d0-40ab-baf2-68738e2966ac # AllViewerExceptHostHeader
            ResponseHeadersPolicyId: 67f7725c-6f97-4210-82d7-5512b31e9d03 # SecurityHeadersPolicy
            ViewerProtocolPolicy: redirect-to-https
      ....
```

These changes are focused on adding a new origin and mapping `/api` to the new origin.

After making these updates, let's update the existing Frontend stack with the new CFN template:

```bash
aws cloudformation update-stack --stack-name your-unique-stack-name --template-body file://frontend.yaml \
  --parameters ParameterKey=APIEndpoint,ParameterValue=xxxxxxx.execute-api.eu-central-1.amazonaws.com
```

After the successful update, try reaching the `/api` endpoint using the CloudFront URL:

```bash
curl https://xxxxxxxxxx.cloudfront.net/api/greeting
```

You should receive a successful response:

```json
{"id":8,"content":"Hello, World!"}
```

Congratulations! You have successfully connected the Frontend and Backend.

However, your backend is currently vulnerable, lacking an authentication layer.

## Protecting Backend with Identity Provider (Cognito)

In this section, we'll integrate AWS Cognito with API Gateway to protect the backend. The main goal is to reject all requests except those with an Authorization Header containing a JWT issued by AWS Cognito.

To achieve this goal, I made changes to the `backend.yaml`:

```yaml
Resources:
...
  ApiGatewayAuthorizer:
    Type: AWS::ApiGateway::Authorizer
    Properties:
      Name: !Sub ${AWS::StackName}-authorizer-backend-proxy
      Type: COGNITO_USER_POOLS
      IdentitySource: method.request.header.Authorization
      RestApiId: !Ref ApiGatewayRestApi
      ProviderARNs:
        - !Ref CognitoPoolArn
...
  ApiGatewayMethod:
    Type: AWS::ApiGateway::Method
    Properties:
...
      AuthorizationType: COGNITO_USER_POOLS
      AuthorizerId: !GetAtt ApiGatewayAuthorizer.AuthorizerId
```

Let's break down what these changes do:

1. **ApiGatewayAuthorizer**: In this section, we define an authorizer for the Amazon API Gateway. An authorizer controls access to your API methods.
    
    * **Type**: It's set to `COGNITO_USER_POOLS`, indicating that it will use Amazon Cognito user pools for authorization.
        
    * **Name**: This is the name of the authorizer, created using the stack name suffixed with "-authorizer-backend-proxy".
        
    * **IdentitySource**: It defines where in the request the identity information is coming from, in this case, it looks for an "Authorization" header.
        
    * **RestApiId**: A reference to the Rest API associated with the authorizer.
        
    * **ProviderARNs**: An array containing the Amazon Resource Name (ARN) of the Cognito user pool. It references another resource, `CognitoPoolArn`, which should be defined elsewhere in the CloudFormation template.
        
2. **ApiGatewayMethod**: Here, we define a method for the API Gateway.
    
    * **AuthorizationType**: It's set to `COGNITO_USER_POOLS`, indicating that the method will use Cognito user pools for authorization. It corresponds to the authorization type defined in the authorizer.
        
    * **AuthorizerId**: The ID of the authorizer used to authorize requests to this method. This value is retrieved from the `ApiGatewayAuthorizer` resource defined earlier.
        

In summary, this CloudFormation script defines an API Gateway method that uses a Cognito user pool for authorization. Users attempting to call this API method must present a valid token from the corresponding Cognito user pool in the Authorization header of their request.

To deploy these updates, run the following command:

```bash
aws cloudformation deploy \
    --stack-name one-click-backend \
    --template-file backend.yaml \
    --parameter-overrides CognitoPoolArn=$cognitoPoolArn ContainerURI=$containerUrl \
    --capabilities CAPABILITY_IAM
```

# Testing the Final Product

Use your CloudFront URL to access the website part of your application. You should see "Login" and "Fetch Data" buttons.

Open the developer console and click "Fetch data." You should receive an error, because JWT is not found.

Now, click "login" and go through the login process. A successful login should result in a lot of data being printed to your console, it's your JWT.

The `id_token` is the value that will be used to query our protected backend.

Click "Fetch data" one more time, and you should receive data from your backend on your HTML page: {"id":9,"content":"Hello, World!"}

Well done! We're almost ready with all the components for a production-ready CFN template.

## Summary

In this article, we demonstrated how to securely connect authentication, backend, and frontend components using a CloudFormation template for a Single Page Application (SPA). We covered updates to the Frontend and Backend templates, the integration of AWS Cognito with API Gateway for authorization, and testing the final product. By the end of this tutorial, you should have a secure and nearly production-ready SPA setup.

In the next part, we'll present your final script that can perform a "one-click" deployment of our application.