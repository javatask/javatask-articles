---
title: "Deploying a Single Page Application (SPA) on AWS: A Beginner's Guide. Part 6. One Click"
seoTitle: "SPA Deployment on AWS: Beginner's Guide Pt.6"
seoDescription: "Simplify AWS deployment using one-click bash script for OpenID, Backend, and Frontend, saving time and easing environment management"
datePublished: Sun Nov 26 2023 16:00:12 GMT+0000 (Coordinated Universal Time)
cuid: clpfo28sp00030alf1qjqe7rs
slug: deploying-a-single-page-application-spa-on-aws-a-beginners-guide-part-6-one-click
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697637595523/a6437798-fdec-4725-8a83-ea6550c29dda.png
tags: cloudformation, aws, bash, ci-cd, iac

---

## Introduction

In the world of software development, managing different environments is crucial. It's not enough to have just a single environment for development; you often need staging, production, end-to-end (E2E) testing, and custom environments for various use cases. However, manually deploying and configuring these environments can be time-consuming and error-prone.

In this article, we'll introduce a single bash script that streamlines the deployment of the entire stack, including the OpenID Server, Backend, and Frontend components. This script automates the process and significantly reduces deployment time.

# Source code repository

[javatask/aws-spa-beginners-guide: Code for the blog series on porting single page application to AWS (](https://github.com/javatask/aws-spa-beginners-guide)[github.com](http://github.com)[)](https://github.com/javatask/aws-spa-beginners-guide)

## The Deployment Script

The deployment script, named `one-click-deploy.sh`, is designed to simplify the deployment process. Here's how it works:

```bash
#!/bin/bash
# Please provide a single parameter to the script, 
# the container URL. 
# Example ./one-click-deploy.sh 11111111.dkr.ecr.eu-central-1.amazonaws.com/xxx:spring-rest-api
export AWS_DEFAULT_REGION=eu-central-1
echo "AWS Region is $AWS_DEFAULT_REGION"

###
# Check if the number of parameters is not equal to 1
if [ "$#" -ne 1 ]; then
  echo "Usage: $0 <container-url>"
  echo "Please provide a single parameter, the container URL. Example 11111111.dkr.ecr.eu-central-1.amazonaws.com/xxx:spring-rest-api"
  exit 1
fi
containerUrl="$1"
# You can add more code here to work with the container_url variable
echo "Container URL provided: $containerUrl"

###
echo Deploing initial AWS Cognito User Pool
aws cloudformation deploy \
    --stack-name one-click-openid \
    --template-file ./templates/openid.yaml \
    --parameter-overrides CallbackUrl=https://example.com/callback

cognitoPoolArn=$(aws cloudformation describe-stacks --stack-name one-click-openid --query 'Stacks[0].Outputs[?OutputKey==`CognitoPoolArn`].OutputValue' --output text)
echo "Cognito Pool ARN $cognitoPoolArn"

cognitoPoolId=$(aws cloudformation describe-stacks --stack-name one-click-openid --query 'Stacks[0].Outputs[?OutputKey==`CognitoPoolId`].OutputValue' --output text)
echo "Cognito Pool Id $cognitoPoolId"

oAuthClientId=$(aws cloudformation describe-stacks --stack-name one-click-openid --query 'Stacks[0].Outputs[?OutputKey==`OAuthClientId`].OutputValue' --output text)
echo "OAuth Client Id $oAuthClientId"

###
echo Adding user named test with password password
aws cognito-idp admin-create-user --user-pool-id $cognitoPoolId --username test --temporary-password password --user-attributes Name=email,Value=name@example.com

###
echo Deploing Backend
aws cloudformation deploy \
    --stack-name one-click-backend \
    --template-file ./templates/backend.yaml \
    --parameter-overrides CognitoPoolArn=$cognitoPoolArn ContainerURI=$containerUrl \
    --capabilities CAPABILITY_IAM

apiDomain=$(aws cloudformation describe-stacks --stack-name one-click-backend --query 'Stacks[0].Outputs[?OutputKey==`ApiGatewayDomain`].OutputValue' --output text)

apiStage=$(aws cloudformation describe-stacks --stack-name one-click-backend --query 'Stacks[0].Outputs[?OutputKey==`ApiGatewayStage`].OutputValue' --output text)

echo "apiUrl https://${apiDomain}/${apiStage}"

###
echo Deploing Frontend
aws cloudformation deploy \
    --stack-name one-click-frontend \
    --template-file ./templates/frontend.yaml \
    --parameter-overrides APIEndpoint=$apiDomain APIEndpointStage=$apiStage

websiteUrl=$(aws cloudformation describe-stacks --stack-name one-click-frontend --query 'Stacks[0].Outputs[?OutputKey==`URL`].OutputValue' --output text)

s3bucket=$(aws cloudformation describe-stacks --stack-name one-click-frontend --query 'Stacks[0].Outputs[?OutputKey==`BucketWithStaticAssets`].OutputValue' --output text)

echo "Website URL: $websiteUrl"

###
echo Updating Cognito callback URL
aws cloudformation deploy \
    --stack-name one-click-openid \
    --template-file ./templates/openid.yaml \
    --parameter-overrides CallbackUrl="$websiteUrl/callback.html"

###
echo Building website
cd frontend

cat > src/settings.js << EOL
export const settings = {
    authority: 'https://cognito-idp.eu-central-1.amazonaws.com/$cognitoPoolId',
    client_id: '$oAuthClientId',
    redirect_uri: '$websiteUrl/callback.html',
    response_type: 'code',
    scope: 'openid',
    revokeTokenTypes: ["refresh_token"],
    automaticSilentRenew: false,
};
EOL
echo "settings.js file has been created"

npm i
npm run build
aws s3 sync dist/ s3://$s3bucket
cd ..
```

**Key Points:**

* The script begins by setting the AWS default region to `eu-central-1` and echoes it.
    
* It checks if the correct number of command-line parameters (in this case, one parameter, the container URL) has been supplied. If not, it prints usage instructions and exits with an error code.
    
* The provided container URL is captured and echoed.
    

The script proceeds to deploy various AWS resources, such as the OpenID Server, Backend, and Frontend, by calling AWS CloudFormation templates and AWS CLI commands. We'll break down the significant steps of the script:

### Deploying AWS Cognito User Pool

```bash
echo Deploying initial AWS Cognito User Pool

# Deploy the Cognito User Pool using CloudFormation
aws cloudformation deploy \
    --stack-name one-click-openid \
    --template-file ./templates/openid.yaml \
    --parameter-overrides CallbackUrl=https://example.com/callback

# Retrieve Cognito information from the stack outputs
cognitoPoolArn=$(aws cloudformation describe-stacks --stack-name one-click-openid --query 'Stacks[0].Outputs[?OutputKey==`CognitoPoolArn`].OutputValue' --output text)
echo "Cognito Pool ARN $cognitoPoolArn"

# Similar lines to retrieve Cognito Pool ID and OAuth Client ID
```

* This section deploys the initial AWS Cognito User Pool using a CloudFormation template. It provides a callback URL as a parameter.
    
* It then retrieves crucial Cognito information such as the ARN, Pool ID, and OAuth Client ID from the stack outputs.
    

### Adding a Test User to Cognito

```bash
echo Adding user named test with password password

# Use AWS CLI to create a test user with a temporary password
aws cognito-idp admin-create-user --user-pool-id $cognitoPoolId --username test --temporary-password Private-1234 --user-attributes Name=email,Value=name@example.com
```

* This part adds a test user named "test" with a temporary password and an email attribute to the Cognito User Pool.
    

### Deploying Backend

```bash
echo Deploying Backend

# Deploy the backend stack using CloudFormation
aws cloudformation deploy \
    --stack-name one-click-backend \
    --template-file ./templates/backend.yaml \
    --parameter-overrides CognitoPoolArn=$cognitoPoolArn ContainerURI=$containerUrl \
    --capabilities CAPABILITY_IAM

# Retrieve API information from the stack outputs
apiDomain=$(aws cloudformation describe-stacks --stack-name one-click-backend --query 'Stacks[0].Outputs[?OutputKey==`ApiGatewayDomain`].OutputValue' --output text)
apiStage=$(aws cloudformation describe-stacks --stack-name one-click-backend --query 'Stacks[0].Outputs[?OutputKey==`ApiGatewayStage`].OutputValue' --output text)
```

* This section deploys the backend stack using a CloudFormation template. It provides parameters like the Cognito Pool ARN and the container URL.
    
* It retrieves API information, including the API Gateway domain and stage, from the stack outputs.
    

### Deploying Frontend

```bash
echo Deploying Frontend

# Deploy the frontend stack using CloudFormation
aws cloudformation deploy \
    --stack-name one-click-frontend \
    --template-file ./templates/frontend.yaml \
    --parameter-overrides APIEndpoint=$apiDomain APIEndpointStage=$apiStage
```

* This part deploys the frontend stack using another CloudFormation template. It takes parameters related to the API endpoint and stage.
    

### Updating Cognito Callback URL

```bash
echo Updating Cognito callback URL

# Update the Cognito callback URL with the frontend URL
aws cloudformation deploy \
    --stack-name one-click-openid \
    --template-file ./templates/openid.yaml \
    --parameter-overrides CallbackUrl="$websiteUrl/callback.html"
```

* This section updates the Cognito callback URL with the URL of the deployed frontend.
    

### Building and Deploying the Website

```bash
echo Building website

# Change the directory to the frontend folder
cd frontend

# Create a settings.js file with Cognito configurations
cat > src/settings.js << EOL
export const settings = {
    authority: 'https://cognito-idp.eu-central-1.amazonaws.com/$cognitoPoolId',
    client_id: '$oAuthClientId',
    redirect_uri: '$websiteUrl/callback.html',
    response_type: 'code',
    scope: 'openid',
    revokeTokenTypes: ["refresh_token"],
    automaticSilentRenew: false,
};
EOL

# Install dependencies and build the website
npm i
npm run build

# Sync the built files to the S3 bucket
aws s3 sync dist/ s3://$s3bucket

# Change back to the script's directory
cd ..
```

* In this section, the script enters the `frontend` directory.
    
* It generates a `settings.js` file with the necessary Cognito configurations.
    
* Dependencies are installed, and the website is built using npm.
    
* The built files are synchronized with an S3 bucket.
    
* Finally, the script returns to its original directory.
    

### Script Execution Time

The script concludes by displaying the real execution time, user time, and system time. This information can be useful for tracking the script's performance.

## Deployment Time

To assess the deployment time, the `time` command is used to execute the script and measure the time it takes. Here's an example output:

```bash
real    11m
user    0m9.5s
sys     0m1.5s
```

* The `real` time represents the total time taken for the script to execute, which in this case is
    

approximately 11 minutes

* `user` time denotes the amount of CPU time spent in user-mode, which is approximately 9.5 seconds.
    
* `sys` time signifies the CPU time spent in kernel-mode, which is approximately 1.5 seconds.
    

The entire deployment process took around 11 minutes, making it a significantly faster and more efficient way to set up complex environments. However, the deployment time can vary depending on factors such as the complexity of the infrastructure and network conditions.

## Summary

In this article, we introduced a powerful bash script designed to automate the deployment of AWS resources for a web application with authentication and backend services. This script streamlines the deployment process and significantly reduces the time required to set up multiple environments.

By breaking down the script into key sections, we provided a comprehensive understanding of its functionality. The script is a valuable tool for managing complex deployment scenarios and offers substantial time savings for development teams.