---
title: "An efficient way to build your serverless microservices. Part 1.  Local Development."
seoTitle: "An efficient way to build your serverless microservices"
seoDescription: "Improving NodeJS Dev UX with AWS SAM, Jest and esbuild. As an example I took GraphQL server for serverless API."
datePublished: Sat Jan 07 2023 10:03:11 GMT+0000 (Coordinated Universal Time)
cuid: clcls2z8m000n08jo1ain7bon
slug: an-efficient-way-to-build-your-serverless-microservices-part-1-local-development
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1673087986930/d8bdc77f-9f07-4623-96e2-9a5413a37a8f.jpeg
tags: lambda, aws, nodejs, graphql, serverless

---

### Level:

Advanced

### Source code:

https://github.com/javatask/graphql-serverless

# The Goal of this series of articles
I had worked with AWS lambdas from days when there was no AWS SAM, and it was the early days of [Serverless Frameowork](https://www.serverless.com/). It was an interesting time to find the best tools and approaches to building serverless applications.
I started with bash, webpack, npm .npmrc configs variables and console.log :)
But after all these years, the industry found better, not yet perfect :), ways to develop serverless apps.
In this series, I'll share the NodeJS project boilerplate that shows state-of-the-art tooling that can be used to develop, test, deploy, test, log, trace, monitor, upgrade any serverless business logic. 

# Series

* [An efficient way to build your serverless microservices. Part 1. Local Development](https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-1-local-development)
    
* [An efficient way to build your serverless microservices. Part 2. Development in the Cloud](https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-2-development-in-the-cloud)
    
* [An efficient way to build your serverless microservices. Part 3. CI/CD with AWS SAM](https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-3-cicd-with-aws-sam)

# Introduction
Building the lifecycle for NodeJS Backend for Frontend (BFF) with good Dev user experiences (UX) is challenging. But to add an additional requirement to use serverless stack and hopefully, it is still possible to prevail.

> ***Note.*** This article is not targeting AWS AppSync. This article uses "raw" AWS API Gateway and AWS Lambda to build GraphQL BFF. AWS AppSync is not used in our use cases because there is a need to have an on-prem version of BFF.



# Prerequirements
Knowledge of

* [NodeJS] (https://nodejs.org/)
    
* [AWS Lambda] (https://aws.amazon.com/lambda/)
    
* [AWS Cloudformation] (https://aws.amazon.com/cloudformation/)
    

# Quick start

For those who want to go directly to code, please [follow https://github.com/javatask/graphql-serverless](https://github.com/javatask/graphql-serverless)   
To run the function, you need to install dependencies on your machine:

* [NodeJS] (https://nodejs.org/) - recommended way to install is [`nvm`](https://github.com/nvm-sh/nvm)
    
* [Docker] (https://www.docker.com/)
    
* [AWS CLI] (https://aws.amazon.com/cli/)
    
* [AWS SAM] (https://aws.amazon.com/sam/)   
And then run commands in your terminal:
```bash 
git clone https://github.com/javatask/graphql-serverless.git 
cd graphql-serverless 
npm i
npm test 
npm run local-run
```

Part 2. Development in the Cloud
[This article guides developers through the deployment and live coding capabilities of AWS SAM](https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-2-development-in-the-cloud)
https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-2-development-in-the-cloud


**Note.** At the end of article there are changes to boilerplate for support of source maps
    

*Note.* These commands should work on Linux or MacOS

# Criteria for good serverless Dev UX

There is need to account for list of features like:

* Language support: [NodeJS 18](https://nodejs.org/)
    
* Language Features: [ES modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
    
* Infrastructure: [IaC (Infrastructure as Code)](https://en.wikipedia.org/wiki/Infrastructure_as_code)
    
* Tests framework: [Jest](https://jestjs.io/) without babel
    
* Local development: [Docker](https://www.docker.com/)
    
* Run and Debug in Cloud environment: [Serverless](https://en.wikipedia.org/wiki/Serverless_computing)
    
* Multi-stage environment support [CI/CD pipelines](https://en.wikipedia.org/wiki/Continuous_integration) (will be covered in other articles)
    
* \[KISS principle\] (https://en.wikipedia.org/wiki/KISS\_principle) - keep it simple stupid
    

After defining the criteria, there is need to check offered implementation offered in https://github.com/javatask/graphql-serverless.

# Local development of Lambdas

## Tooling and package.json

```json
{
...
  "type": "module",
...
  "dependencies": {
    "@apollo/server": "^4.3.0",
    "@as-integrations/aws-lambda": "^1.1.0",
    "graphql-tag": "^2.12.6"
  },
  "devDependencies": {
    "esbuild": "0.16.12",
    "jest": "^29.2.1"
  },
  "scripts": {
    "test": "node --experimental-vm-modules ./node_modules/jest/bin/jest.js",
    "local-run": "npm run build && sam local invoke --event ./events/graphql-post.json --env-vars env.json GraphqlServerFunction",
    "local-api": "npm run build && sam local start-api --env-vars env.json",
    "build": "sam build",
    "deploy": "sam deploy --guided",
    "build-deploy": "npm run build && npm run deploy",
    "cloud-watch": "sam sync --stack-name graphql-serverless --watch",
    "cloud-logs": "sam logs -n GraphqlServerFunction --stack-name graphql-serverless"
  }
...
}
```

### ESM

To enable ESM by default, there is directive `"type":"module"` in `package.json`, it will instruct node to treat all js files as ESM.

### Dependencies section

* *@apollo/server*: This is the Apollo Server package, which is a popular GraphQL server for Node.js.
    
* *@as-integrations/aws-lambda*: This is the AWS Lambda integration for Apollo Server. It allows you to deploy an Apollo Server instance as an AWS Lambda function.
    
* *graphql-tag*: This is a utility package for parsing GraphQL queries.
    

And Dev dependencies

* *esbuild*: This is a JavaScript and TypeScript bundler and minifier. It can be used to build and optimize the project for production.
    
* *jest*: This is a popular test runner for JavaScript projects. It can be used to run and automate tests.
    

### Scripts section

Scripts sections define next scripts:

* *test*: This script runs the test suite for the project. It uses the jest test runner to execute the tests. There are two tests: 1)First test for business logic; 2) Second test for GraphQL query
    
* *local-run*: This script builds the project and invokes the AWS Lambda function locally using the `sam local invoke` command. It passes the event data from the `./events/graphql-post.json` file and the environment variables from the `env.json` file.
    
* *local-api*: This script builds the project and then starts the local API Gateway using the `sam local start-api` command. It uses the environment variables from the `env.json` file. After running this command, you can query the local instance of the server with the next command 
```bash 
curl --request POST 
--url http://127.0.0.1:3000/graphql 
--header 'Content-Type: application/json' 
 --data '{"query":"query Test {state}"}'
```
    
* *build*: This script builds the project using the `sam build` command.
    
* *deploy*: This script deploys the project to AWS using the `sam deploy` command. To deploy the stack, you need to pass your `samconfig.toml` file or use `--guided` key with this command
    
* *build-deploy*: This script runs the build and deploys scripts consecutively.
    
* *cloud-watch*: This script syncs the local file changes with the stack running in the cloud and restarts the function if needed using the `sam sync` command.
    
* *cloud-logs*: This script fetches the logs for the specified AWS Lambda function using the `sam logs` command.
    

### Tips and tricks for local development

When writing this article, there was no easy way to watch code changes while running the `npm run local-api` command. To update code for your local running server, you need to run the `npm run build` command and do NOT restart `npm run local-api` command. I recommend running two terminal windows, one running `npm run local-api` server and the second running `npm run build` command after code changes.

But the better approach is to use `jest --watch` to develop your business logic. You are then using `jest` to test your GraphQL revolvers and queries. Finally, after good code coverage is reached, run your local server to do your GraphQL service integration tests.

> *Note.* Local server also can be used to implement integration tests with other parts of your system

> *Note.* What if I need to use AWS service X.Y.Z. for business logic? In this case I can recommend using [Hexagonal architecture](https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)) to mock this services with [Jest mocking user modules](https://jestjs.io/docs/manual-mocks#mocking-user-modules)

**Congratulations!!! Your lambda works locally!!! **


# Summary

AWS Serverless Application Model (SAM) gives many tools to improve developer user experiences while developing serverless microservices. Among these features are:

* support of latest LTS NodeJS
    
* integration with state-of-the-art JavaScript bundler - [esbuild](https://esbuild.github.io/)
    
* support for local invocation of Lambdas
    
* support for local invocation of API Gateway
  

This article references the code repository https://github.com/javatask/graphql-serverless and illustrates all these features. Including Jest configuration.

# Part 2. Development in the Cloud
[This article guides developers through the deployment and live coding capabilities of AWS SAM](https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-2-development-in-the-cloud)
https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-2-development-in-the-cloud 