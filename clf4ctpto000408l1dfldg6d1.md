---
title: "An efficient way to build your serverless microservices. Part 3. CI/CD with AWS SAM."
datePublished: Sat Mar 11 2023 19:23:07 GMT+0000 (Coordinated Universal Time)
cuid: clf4ctpto000408l1dfldg6d1
slug: an-efficient-way-to-build-your-serverless-microservices-part-3-cicd-with-aws-sam
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678461424256/ed835e20-a4ad-494e-b8fb-10bf5aac8e8c.jpeg
tags: aws, testing, serverless, cicd, github-actions-1

---

# Introduction

Continuous Integration/Continuous Deployment (CI/CD) is essential to modern software development. With the help of CI/CD pipelines, developers can automate their software delivery processes, reducing the risk of human errors and increasing productivity. This article in [An efficient way to build your serverless microservices](https://blog.javatask.dev/series/awssam) series will look at generating a CI/CD pipeline using AWS Serverless Application Model (SAM), Node.js, and GitHub Actions.

# Series

* [An efficient way to build your serverless microservices. Part 1. Local Development](https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-1-local-development)
    
* [An efficient way to build your serverless microservices. Part 2. Development in the Cloud](https://blog.javatask.dev/an-efficient-way-to-build-your-serverless-microservices-part-2-development-in-the-cloud)
    
* An efficient way to build your serverless microservices. Part 3. CI/CD with AWS SAM
    

# Quick start

The whole pipeline is defined in [pipeline.yaml](https://github.com/javatask/graphql-serverless/blob/main/.github/workflows/pipeline.yaml) using [GitHub Actions](https://github.com/features/actions). The first step is to run `sam pipeline bootstrap` to get values for:

* `TESTING_PIPELINE_EXECUTION_ROLE` + `PROD_PIPELINE_EXECUTION_ROLE`
    
* `TESTING_CLOUDFORMATION_EXECUTION_ROLE` + `PROD_CLOUDFORMATION_EXECUTION_ROLE`
    
* `TESTING_ARTIFACTS_BUCKET` + `PROD_ARTIFACTS_BUCKET`
    

The next step is setting [GitHub Repository Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` that provide access to your AWS account. The last step is to push changes in `pipeline.yaml` to your GitHub repository. A new commit will trigger the creation of test and production environments using Cloudfromation stacks.

## What is AWS Serverless Application Model (SAM)?

AWS Serverless Application Model (SAM) is an open-source framework for building serverless AWS applications. SAM provides a simplified way to define the Amazon API Gateway APIs, AWS Lambda functions, and Amazon DynamoDB tables your serverless application needs. SAM is built on top of AWS CloudFormation, so all resources defined in the SAM template are created and managed by CloudFormation.

## What is Node.js?

Node.js is an open-source, cross-platform, back-end JavaScript runtime environment that executes JavaScript code outside a web browser. Node.js enables developers to build scalable, high-performance applications using JavaScript on the server side.

## What are GitHub Actions?

GitHub Actions is a powerful tool that allows developers to automate their software development workflows. With GitHub Actions, developers can create custom workflows triggered by specific events, such as a pull request or a code push. Now that we know what AWS SAM, Node.js, and GitHub Actions are, let's look at how to use them to create a CI/CD pipeline.

# Prerequisites:

Before we start, you will need to have the following:

* An AWS account
    
* Node.js installed on your local machine
    
* A GitHub account
    
* AWS CLI
    
* AWS SAM
    

# Generate CI/CD pipeline

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678523984575/a324f1cd-6cb4-4fd5-87ee-6b2762ba0cea.png align="center")

The latest AWS SAM version has capabilities to generate [CI/CD](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-generating-example-ci-cd.html) pipelines for:

* Jenkins
    
* GitLab CI/CD
    
* GitHub Actions
    
* Bitbucket Pipelines
    
* AWS CodePipeline
    

But please be aware that it is your starting point for further configuration. Generate CI/CD pipeline has the following features:

* two staged, e.g. dev and prod
    
* includes a dedicated pipeline for [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)
    
* has placeholders to run unit test
    
* has placeholders to run integration test
    

> **Note.** Feature branch workflow in CI/CD suggests deploying dedicated infrastructure to play with the feature. After deleting the feature branch, the dedicated infrastructure is deleted.

## Details for our GraphQL server example

Before we start the generation of CI/CD configuration, I want to share additional changes that I will make to the sample GraphQL server project:

* adding `esbuild` bundler to pack NodeJS code
    
* launching `unit tests` before deploying infrastructure
    
* launching `integration tests` after the deployment of the infrastructure
    

### Esbuild in CI/CD pipeline

During the configuration of a pipeline, I found out that `esbuild` is not included in [`sam deploy action`](https://github.com/marketplace/actions/sam-deploy-action). Without `esbuild` our build step will fail, so the fix was to move `esbuild` from `devDependencies` sections to `dependencies` section. The final `package.json` file looks like

```json
  "dependencies": {
    "esbuild": "0.17.11"
  },
  "devDependencies": {
    "jest": "^29.4.3"
  },
```

### Unit tests

There is not much talk about the test section. It uses the standard `test` command from `package.json`.

```json
"scripts": {
    "test": "node --experimental-vm-modules ./node_modules/jest/bin/jest.js",
```

### Integration test

For the integration test implementation, I took a tool recommended by [Thoughtworks Technology Radar](https://www.thoughtworks.com/en-de/radar/tools/k6) tool called `k6`. `k6` uses javascript syntax to write test cases, but be aware - it's NOT javascript. Our test script looks like this.

```javascript
import http from 'k6/http';
import { check } from 'k6';

export default function () {
  const query = `query Test { state }`;
  const url = `${__ENV.GRAPHQL_URL}/graphql`;
  const headers = { 'Content-Type': 'application/json' };
  const body = JSON.stringify({ query });

  const res = http.post(url, body, { headers });

  check(res, {
    'status is 200': (r) => r.status === 200,
    'response contains state': (r) => r.body.includes('state'),
  });
}
```

This is a k6 script that sends a GraphQL query to a server and verifies the response. Here is a breakdown of the script:

1. The script imports the necessary k6 libraries: http and check.
    
2. The script defines the main`function export default function () {...}` which k6 will execute during the load test.
    
3. The function defines a GraphQL query as a string variable named query: `const query = query Test { state };`. The query requests the state field's value from the GraphQL server.
    
4. The function defines the URL of the GraphQL server as a string variable named url: `const url = ${__ENV.GRAPHQL_URL}/graphql;`. The value of the URL is read from an environment variable named `GRAPHQL_URL`.
    
5. The function defines a JSON header as a JavaScript object named headers: `const headers = { 'Content-Type': 'application/json' };`. This header tells the server that the request body is in JSON format.
    
6. The function defines a JSON request body as a string variable named body: `const body = JSON.stringify({ query });`. The request body is a JSON object that contains the GraphQL query string as a property named `query`.
    
7. The function sends a HTTP POST request to the GraphQL server with the http.post method: `const res = http.post(url, body, { headers });`. The method takes three arguments: the URL of the server, the request body, and the request headers.
    
8. The function uses the check method to verify that the response from the server has a status code of 200 and contains the word 'state'. This is done to ensure that the server is responding correctly and returning the expected data.
    

## Bootstrapping CI/CD

The first step in generating CI/CD pipeline is getting roles for your Cloudfromation stack. This step is described in [AWS SAM documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-generating-example-ci-cd.html):

> Here are the high-level tasks you need to perform to generate a starter pipeline configuration:
> 
> 1. Create infrastructure resources – Your pipeline requires certain AWS resources, for example the IAM user and roles with necessary permissions, an Amazon S3 bucket, and optionally an Amazon ECR repository.
>     
> 2. Connect your Git repository with your CI/CD system – Your CI/CD system needs to know which Git repository will trigger the pipeline to run. Note that this step may not be necessary, depending on which combination of Git repository and CI/CD system you are using.
>     
> 3. Generate your pipeline configuration – This step generates a starter pipeline configuration that includes two deployment stages.
>     
> 4. Commit your pipeline configuration to your Git repository – This step is necessary to ensure your CI/CD system is aware of your pipeline configuration, and will run when changes are committed.
>     

It would be best if you run the `Create infrastructure resources` step once to create the required AWS resources for CI/CD. To initiate this step please run

```bash
sam pipeline bootstrap
```

and follow the wizard.

## Configuration of CI/CD pipeline

The next step is to `Generate your pipeline configuration` performed via

```bash
sam pipeline init
```

The result of this command is `.github/workflows/pipeline.yaml` file with the initial pipeline configuration. This pipeline has placeholders for `unit` and `integration` tests. Let's cover the required changes to "enable" the test section.

### Unit tests

To run `nodejs` unit tests, there is a need to change 41 LoC from

```yaml
  test:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: |
          # trigger the tests here
```

to

```yaml
test:
    if: github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - name: Install npm dependencies
        run: npm install
      - run: |
          npm test
```

I added nodejs installation to the step and installed project dependencies, including `jest` and command to launch the `test` script from the `package.json`

### Integration tests

To enable integration tests, there is a need to perform more actions. The main issue is getting the publically available URL of our newly created graphql server. The first change is in `deploy-testing` block, on LoC 218

```yaml
 - name: Get GraphQL endpoint from testing stack
        run: |
          GRAPHQL_CF_URL=$(aws cloudformation describe-stacks --stack-name ${TESTING_STACK_NAME} --query 'Stacks[0].Outputs[?OutputKey==`WebEndpoint`].OutputValue' --output text)
```

Here I added a step to create a new environment variable, `GRAPHQL_CF_URL` with the public graphql server URL value extracted from the Output values of the Cloudformation template.

> *Note.* You need explicitly put the Output value into the Cloudformation template to use the same technique. The following change was related to `integration-test` block, I did change from

```yaml
integration-test:
    if: github.ref == 'refs/heads/main'
    needs: [deploy-testing]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: |
          # trigger the integration tests here
```

to

```yaml
integration-test:
    if: github.ref == 'refs/heads/main'
    needs: [deploy-testing]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run k6 local test
        uses: grafana/k6-action@v0.2.0
        env:
          GRAPHQL_URL: $GRAPHQL_CF_URL
        with:
          filename: integration/load.js
```

Here I added `grafana/k6-action@v0.2.0` action that runs k6 integration test script with `GRAPHQL_CF_URL` as parameter.

### AWS Secret configuration

Before committing the pipeline to GitHub please follow [GitHub Repository Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) guide to configure repository secrets named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` that provide access for CI/CD to your AWS account.

## Result

Figure 1. shows the result of changes to the boilerplate pipeline. After a couple of changes to AWS SAM CI/CD script, you got a production-ready CI/CD pipeline for your serverless application.

![Result of Ci/CD pipeline](https://cdn.hashnode.com/res/hashnode/image/upload/v1678460601572/52022d42-c279-4048-8023-3f355ac499b5.png align="center")

Figure 1. Result of running CI/CD pipeline

# Summary

AWS SAM helps configure production-ready CI/CD pipelines that follow best practices for securing your Cloudfromation deployments and incorporating unit and integration tests into your CI/CD pipeline.

# Final thoughts

In this series, I tried to show the whole software lifecycle for developing serverless applications using AWS SAM. My three articles (develop, deploy and automate) described easy and convenient to start developing your serverless applications. Serverless differs from a developer experience perspective and has slightly different development and testing tools and techniques. Also, this article tried to be highly practical without diving into details of concepts like:

* (Hexagon architecture)\[https://en.wikipedia.org/wiki/Hexagonal\_architecture\_(software)\]
    
* (Microservices)\[https://en.wikipedia.org/wiki/Microservices\]
    
* (Enterprise Integration Patterns)\[https://en.wikipedia.org/wiki/Enterprise\_Integration\_Patterns\]
    
* (Distributed computing)\[https://en.wikipedia.org/wiki/Distributed\_computing\]
    
* etc
    

I defiantly recommend to the reader to balance the practical usage of serverless applications and reading through the theory behind the serverless technology stack.