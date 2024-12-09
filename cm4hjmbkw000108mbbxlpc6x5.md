---
title: "How to Define AI Agents with Cloudformation and SAM: A Builder's Guide"
seoTitle: "Building AI Agents with CloudFormation & SAM"
seoDescription: "Discover how to define AI Agents using CloudFormation and AWS SAM for scalable deployment. Learn key components, configurations, and best practices"
datePublished: Mon Dec 09 2024 21:26:58 GMT+0000 (Coordinated Universal Time)
cuid: cm4hjmbkw000108mbbxlpc6x5
slug: how-to-define-ai-agents-with-cloudformation-and-sam-a-builders-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1732109917516/d247055f-d53e-4158-8238-6a36c9f44213.jpeg
tags: aws, data-science, learning, serverless, generative-ai

---

## Introduction

In my previous articles, we explored how AI Agents can transform business operations [from a bakery CEO's perspective](https://blog.javatask.dev/what-c-level-leaders-need-to-know-about-ai-agents) and [showcased data workflow](https://blog.javatask.dev/developing-an-ai-intern-for-c-level-executives-with-aws-bedrock). Let's dive deep into the technical implementation using AWS Bedrock AI Agents defined via CloudFormation and SAM. This guide will focus on the key components of the AI Agent configuration and deployment process, including SAM AWS Lambda to grab data, AI Agent tooling, and AI Agent itself.

This article aims to give you a working template for building multiple custom AI Agents and showcase how to call them programmatically from your custom user interface.

> **Note 1**. If you don’t feel confortable with code I still recommend you to check AI Agent definition to get general understanding of moving parts. It is human readble and I hope logical: [https://github.com/javatask/ai-agent-ceo-fin-advisor/blob/main/bedrock-agent-stack.yaml](https://github.com/javatask/ai-agent-ceo-fin-advisor/blob/main/bedrock-agent-stack.yaml)
> 
> Scroll down to skip permission policy section.
> 
> **Note 2**. Source code is avaliable here: [https://github.com/javatask/ai-agent-ceo-fin-advisor/tree/main](https://github.com/javatask/ai-agent-ceo-fin-advisor/tree/main)

## Prerequisites

Before starting, ensure you have:

* AWS Account with Bedrock model access enabled
    
* AWS CLI configured with appropriate permissions
    
* AWS SAM CLI installed
    
* Basic understanding of CloudFormation and Lambda
    
* JQ was installed for JSON processing, and I used it to grab AWS Lambda arn from SAM deployment
    

## Implementation Overview

Our implementation consists of two main CloudFormation stacks:

1. **Finance Tool Lambda (The Intern's Laptop)**:
    
    * Handles ad-hoc data generation function using SAM
        
    * In production, replace with your data access layer
        
2. **Bedrock AI Agent Configuration (The Intern's Guide)**:
    
    * Contains AI Agent Guidance
        
    * Manages access to tooling
        
    * Defines operational parameters
        

It is best practice to manage the life cycle of your Interns via the [OpenGitOps](https://opengitops.dev/) approach, so any degradation in performance or need to be deployed in other regions can be easily made via CI/CD. Ad-hoc changing tooling, instructions and/or interns are OK for development, but for production release, put the Intern version under git control and deploy via CI/CD pipeline.

## Understanding AI Agent Components

### 1\. Intern Laptop or Action Groups

The AI Agent needs specific tools to perform tasks. In our case, I defined two functions only `analyze_industry_performance` and `send_email`. The goal of `analyze_industry_performance` the function is to take three parameters. `industry`, `date_from` and `date_to` - these parameters are taken from chat input. For example, in my previous article, I gave the Intern a question `How did we do with the hospitality business in the first half of the year?`. Our Intern was capable of mapping `Hospitality` to function parameters `industry=hotels` and setting valid `date_from` and `date_to`. After calling `analyze_industry_performance` function, the Intern can make HTML or Markdown formatting to send a report via email or show it as a Markdown output.

*It is important* to give a detailed description of the function and its parameters so the Intern is capable of mapping CEO intent to function or even asking additional questions that are required to build a report based on CEO intent.

```yaml
ActionGroups:
        - ActionGroupName: financial-analysis-actions
          Description: Financial analysis and reporting functions
          ActionGroupState: ENABLED
          ActionGroupExecutor:
            Lambda: !Ref LambdaFunctionArn
          FunctionSchema:
            Functions:
              - Name: analyze_industry_performance
                Description: |
                  Analyzes financial performance metrics across a network of bakeries, providing insights into bookings, billings, and industry-specific trends.
                  Industry Parameter Values Valid values:
                  'schools' - Educational institutions and campus dining
                  'cafes' - Coffee shops and small eateries
                  'shops' - Retail bakery outlets
                  'factories' - Industrial/manufacturing facilities
                  'restaurants' - Full-service restaurants
                  'hotels' - Hospitality sector
                  Date Range Parameter Format: ('YYYY-MM-DD', 'YYYY-MM-DD')
                  Valid ranges:
                  Full year: date_from:'2024-01-01', date_to:'2024-12-31'
                  Q1: date_from:'2024-01-01', date_to:'2024-03-31'
                  Q2: date_from:'2024-04-01', date_to:'2024-06-30'
                  Custom: Any date range within 2024
            
                Parameters:
                  date_from:
                    Type: string
                    Description: date from in format YYYY-MM-DD
                    Required: true
                  date_to:
                    Type: string
                    Description: date to in format YYYY-MM-DD
                    Required: true
                  industry:
                    Type: string
                    Description: industry for each need to do analysis
                    Required: true
              - Name: send_email
                Description: Function to send an email to CEO with arbitrary payload
                Parameters:
                  html_body:
                    Type: string
                    Description: HTML text of the email
                    Required: true
                  subject:
                    Type: string
                    Description: subject of email
                    Required: true
```

### 2\. Your Guide to Intern or Agent Instructions

Instructions are crucial for AI Agent behaviour. The main point here is to guide the Intern on how to use a laptop and data access.

```yaml
      Instruction: |
        You are an experienced financial analyst specializing in business finance. 
        Your task is to prepare ad-hoc reports for the CEO based on the financial data 
        provided and the specific request made. Follow these instructions carefully 
        to produce a comprehensive and insightful report.

        First, you will be presented with the financial data by using the function analyze_industry_performance
        Next, you need to work on CEO's specific request.
        If required, use the send_email function to send the HTML version of the report to the CEO's email.

        Analyze the financial data in the context of the CEO's request. Consider the following steps:

        1. Identify the key financial metrics relevant to the CEO's request.
        2. Perform necessary calculations and comparisons.
        3. Look for trends, patterns, or anomalies in the data.
        4. Consider both short-term and long-term implications of the findings.

        When preparing your report, adhere to these guidelines:

        1. Be concise yet comprehensive.
        2. Use clear, professional language.
        3. Support your analysis with specific data points from the provided financial information.
        4. Provide actionable insights and recommendations when appropriate.
        5. Anticipate follow-up questions the CEO might have and address them proactively.
```

### 3\. Agent Configuration

Key agent parameters include:

1. Agent Name
    
2. Agent Role - In AWS, everyone should have explicit permission to use the Intern's Laptop; the Agent role explicitly permits the Intern to use a company Laptop and company data.
    
3. The description gives you the ability to navigate through hundreds of Interns
    
4. **FoundationModel -** *Don’t ignore this parameter*, it is the University that your Intern graduated from, it defines it’s “intelligence“, speed and cost. If you find some tasks that are less intelligent but required very frequently, you may hire other Interns via this field.
    

```yaml
FinancialAnalysisAgent:
    Type: AWS::Bedrock::Agent
    Properties:
      AgentName: !Ref AgentName
      AgentResourceRoleArn: !GetAtt BedrockAgentRole.Arn
      Description: Demo of CEO finance assistant
      FoundationModel: arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-5-sonnet-20240620-v1:0
```

## First day on the job on the Deployment Process

Before the Intern joins your team, I assume you already gave it permission to your well-defined and secure [Data Lakehouse](https://aws.amazon.com/sagemaker/lakehouse/), defined tools to query your data and instructions to follow when asked questions.

Let's walk through the onboarding or deployment script:

1. **Build and Deploy Lambda Function**
    

```bash
cd finance-tool-lambda
sam build
sam deploy --stack-name ai-agent-finance-tool-$RANDOM_SUFFIX
```

2. **Deploy AI Agent Stack**
    

```bash
aws cloudformation deploy \
    --template-file bedrock-agent-stack.yaml \
    --stack-name ceo-fin-report-$RANDOM_SUFFIX \
    --parameter-overrides file://agent-params.json \
    --capabilities CAPABILITY_IAM
```

3. **Prepare Agent for Use**
    

```bash
aws bedrock-agent prepare-agent --agent-id $AGENT_ID
```

## First question to the Intern or Testing Your AI Agent

For testing, I’ll use a [simple](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/javascript_bedrock-agent-runtime_code_examples.html) NodeJS script and AWS SDK for JS to ask the same question from the previous article: `How did we do with the hospitality business in the first half of the year? Please send the report to email.`

I assume that you have credentials via `aws sso login` and changed `AgentId` to yours. You can find `AgentId` in the newly deployed Cloudformation stack output or on the AWS Bedrock Agents page.

```javascript
import {
  BedrockAgentRuntimeClient,
  InvokeAgentCommand,
} from "@aws-sdk/client-bedrock-agent-runtime";

/**
 * @typedef {Object} ResponseBody
 * @property {string} completion
 */

/**
 * Invokes a Bedrock agent to run an inference using the input
 * provided in the request body.
 *
 * @param {string} prompt - The prompt that you want the Agent to complete.
 * @param {string} sessionId - An arbitrary identifier for the session.
 */
export const invokeBedrockAgent = async (prompt, sessionId) => {
  const client = new BedrockAgentRuntimeClient({ region: "us-east-1" });
  // const client = new BedrockAgentRuntimeClient({
  //   region: "us-east-1",
  //   credentials: {
  //     accessKeyId: "accessKeyId", // permission to invoke agent
  //     secretAccessKey: "accessKeySecret",
  //   },
  // });

  const agentId = "AJBHXXILZN";
  const agentAliasId = "TSTALIASID";

  const command = new InvokeAgentCommand({
    agentId,
    agentAliasId,
    sessionId,
    inputText: prompt,
  });

  try {
    let completion = "";
    const response = await client.send(command);

    if (response.completion === undefined) {
      throw new Error("Completion is undefined");
    }

    for await (const chunkEvent of response.completion) {
      const chunk = chunkEvent.chunk;
      const decodedResponse = new TextDecoder("utf-8").decode(chunk.bytes);
      completion += decodedResponse;
    }

    return { sessionId: sessionId, completion };
  } catch (err) {
    console.error(err);
  }
};

// Call function if run directly
import { fileURLToPath } from "node:url";
if (process.argv[1] === fileURLToPath(import.meta.url)) {
  const result = await invokeBedrockAgent("How did we do with the hospitality business in the first half of the year? Please send the report to email.", "123");
  console.log(result);
}
```

Launching this script via `node index.mjs` I got a response:

```bash
{
  sessionId: '123',
  completion: 'We had a relatively stable performance in the hospitality business during the first half of the year, with a slight decrease from Q1 to Q2. Here are the key points:\n' +
    '\n' +
    '1. Total bookings for H1 2024 were $282,607.99, with total billings of $255,309.52.\n' +
    '2. There was a small decline in performance from Q1 to Q2:\n' +
    '   - Bookings decreased by 1.51%\n' +
    '   - Billings decreased by 2.76%\n' +
    '3. The average billing rate slightly decreased from $90.92 in Q1 to $89.64 in Q2.\n' +
    '4. May was our strongest month, while June showed the weakest performance.\n' +
    '\n' +
    "I've sent a detailed report to your email with further analysis and recommendations for improving performance in the second half of the year."
}
```

## Conclusion

While this implementation requires traditional development skills, the power lies in decoupling Tools from the Brain. This separation enables:

1. Reuse of tooling across multiple Interns
    
2. Creation of specialized Intern teams
    
3. Complex cross-industry analysis capabilities
    
4. Rapid iteration and improvement cycles
    

Focus on GitOps practices for your AI Agents while trusting your developers with the coding tasks. This approach creates a scalable foundation for unlimited AI workforce expansion.

The complete CloudFormation template and deployment scripts are available in the [GitHub repository](https://github.com/javatask/ai-agent-ceo-fin-advisor).