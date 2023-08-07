---
title: "Migrating your classic Single Page Application to AWS. Part 5"
slug: migrating-your-classic-single-page-application-to-aws-part-5
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1690994014747/7a9ac0f3-4dcb-4a01-834b-3c31172e8a6c.png

---

# Introduction

In this part, I will show you how to connect Authentication, Backend and Frontend bringing a secure SPA Cloudfromation (CFN) template, not yet perfect, but almost production ready. Figure 1 shows all the parts that we will bind together.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1690993456357/91c798b7-820a-4eaa-bb50-812b4fe8961e.png align="center")

Figure 1. All the parts connected

Let's recap the story of the previous parts and the changes that are required to bind them together:

* Part 1. Was an introduction to what we are doing.
    
* Part 2. Frontend. Here I did the deployment of AWS Cloudfront CDN Distribution, and in this part, I will update its CFN template to map Frontend `/api` queries to the Backend.
    
* Part 3. Backend. I configured VM (EC2 Instance) to run REST API Server and API Gateway to proxy requests to VM. In this part, I will also do the changes to protect API Gateway with our OpenID Server JSON Wen Token (JWT). Another important change that I did to the backend is the endpoint change, from `/greeting` to `/api/greeting`
    
* Part 4. Identity Provider. I deployed AWS Cognito as OpenID Server to do the user Authentication. No changes will be made in this part to the initial CFN template.
    

# **Connect Frontend to the Backend**

I'll list here only the change required to the `frontend.yml` file. The whole CFN template you can find in the tutorial folder \[TODO Add URL\].

```yaml
...
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

These changes are focused on adding a new Origin and Mapping `/api` to the new Origin.

Let's update the existing Frontend stack with the new CFN template:

```bash
aws cloudformation update-stack --stack-name your-uniq-stack-name --template-body file://frontend.yaml \
  --parameters ParameterKey=APIEndpoint,ParameterValue=65k7zipnra.execute-api.eu-central-1.amazonaws.com
```

After the successful update, let's try to reach `/api` endpoint using CloudFront URL:

```bash
curl https://xxxxxxxxxx.cloudfront.net/api/greeting
```

You should get a successful response:

```json
{"id":5,"content":"Hello, World!"}
```

My congratulations! You successfully connected Frontend and Backend!

But, your backend is vulnerable, there is no Authentication layer, yet.

# Protecting Backend with Identity Provider (Cognito)

In this section, I'll show you how to integrate AWS Cognito with API Gateway. The core goal is to reject all requests, apart from those which has Authorization Header with JWT value issued by AWS Cognito.

To reach this goal, I made changes to the `backend.yaml` :

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

Let's describe what I did:

1. **ApiGatewayAuthorizer**: At this line, I'm defining an authorizer for an Amazon API Gateway. An authorizer is responsible for controlling access to your API methods.
    
    * **Type**: It's set to `COGNITO_USER_POOLS`, meaning it will use Amazon Cognito user pools for authorization.
        
    * **Name**: The name of the authorizer. It's created using the stack name suffixed with "-authorizer-backend-proxy".
        
    * **IdentitySource**: This defines the location in the request where the identity information is coming from. Here, it's looking for an "Authorization" header.
        
    * **RestApiId**: A reference to the Rest API that the authorizer will be associated with.
        
    * **ProviderARNs**: An array containing the Amazon Resource Name (ARN) of the Cognito user pool. It's referencing another resource, `CognitoPoolArn`, which should be defined elsewhere in the CloudFormation template.
        
2. **ApiGatewayMethod**: Here, I'm defining a method for the API Gateway.
    
    * **AuthorizationType**: This is set to `COGNITO_USER_POOLS`, meaning the method will use the Cognito user pools for authorization. It corresponds with the authorization type defined in the authorizer.
        
    * **AuthorizerId**: The ID of the authorizer that's being used to authorize requests to this method. It's retrieving this value from the `ApiGatewayAuthorizer` resource that was defined earlier.
        

In summary, this CloudFormation script is defining an API Gateway method that uses a Cognito user pool for authorization. Users who want to call this API method will need to present a valid token from the corresponding Cognito user pool in the Authorization header of their request.

To deploy updates run:

```bash
aws cloudformation deploy \
    --stack-name one-click-backend \
    --template-file backend.yaml \
    --parameter-overrides CognitoPoolArn=$cognitoPoolArn ContainerURI=$containerUrl \
    --capabilities CAPABILITY_IAM
```

# Testing final product

Use your CloudFront URL to reach the website part of your app:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691410568141/e2aa5cf6-fa60-4f03-b00d-0b3552482f08.png align="center")

Open the developer console and click `Fetch data`, you should get the error:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691410672034/2d9beba5-d1c3-48ba-8bef-0db954a69b5e.png align="center")

Now click `login` and go through the login process, successful login should print a lot of data into your console:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691410904035/72b4b4df-7f77-4cbd-880c-06954cca57b2.png align="center")

`id_token` is the value that will be used to query our protected backend.

Click `Fetch data` one more time, you should get data from your backend:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1691411025185/b3ec95b1-6e8a-4f3c-829a-03f8f132d738.png align="center")

Well done! We almost have all bits and pieces to get production-ready CFN templates.

# Summary

In this article, I demonstrate how to connect Authentication, Backend, and Frontend securely using a CloudFormation template for a Single Page Application (SPA). We cover updating the Frontend and Backend templates, integrating AWS Cognito with API Gateway for authorization, and testing the final product. By the end of this tutorial, you'll have a secure and nearly production-ready SPA setup.

In the next part, I'll show your final script that can do a "one-click" deployment of our application.