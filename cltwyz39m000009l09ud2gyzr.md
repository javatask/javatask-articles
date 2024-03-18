---
title: "Fixing Issues in AWS VPC with AI Help"
seoTitle: "AI Solutions for AWS VPC Issues"
seoDescription: "Learn how AI simplifies fixing AWS VPC issues, enhancing EC2 access through optimized security configurations. Ideal for AWS networking enthusiasts"
datePublished: Mon Mar 18 2024 13:16:35 GMT+0000 (Coordinated Universal Time)
cuid: cltwyz39m000009l09ud2gyzr
slug: fixing-issues-in-aws-vpc-with-ai-help
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1710767671282/6d1542d6-5abb-4017-b73d-1733152b6169.webp
tags: ai, aws, networking, vpc, generative-ai, amazon-bedrock

---

## Introduction

My story begins with AWS Advance Networking Certification. I will start learning all possible network configurations and buy a notebook to note all VPC, Subnets, Route Tables, ENI, and other entity IDes to debug misconfigurations. I have an idea: All this information is available via AWS CLI. Maybe Generative AI can help me with this?!

So, I built a simple react AI Agent to help with some simple stuff. Maybe this article will inspire you ;)

## Result

If you, my honoured reader, want to understand whether this article is for you, here is the result of the small eight-operator code.

> **Question**: I cannot access web server at ec2 instance ID i-xxxxxx.
> 
> **Answer**: Initially, you were unable to access the web server on the EC2 instance i-xxxxxx because the associated security group did not have an inbound rule permitting web traffic. Specifically, it lacked rules allowing HTTP (port 80) or HTTPS (port 443) inbound connections.  
> !!!  
> To resolve this issue, an inbound rule has been added to the security group (sg-xxxxxx) to permit traffic on port 80 from any IP address (0.0.0.0/0).  
> !!!!  
> With this change, you should now be able to access the web server using the public IP address xxx.xxx.xxx.xxx.  
> However, if you continue experiencing difficulties accessing the web server after adding the security group rule, you may need to investigate further. Potential areas to review include the web server configuration on the EC2 instance, ensuring the web server software is running, and verifying that the instance is located in a public subnet with an Internet Gateway attached to the VPC's route table.

## Prerequisites

1. You should be comfortable with AWS Console, Cloudformation, VPC
    
2. You should configure access to the [AWS Bedrock Calude 3 Sonnet model](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access.html)
    
3. You should have [Project Jupyter | Home](https://jupyter.org/) installed
    
4. Also, I'm using `aws sso login` to secure access to [AWS Enviroment](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html)
    

## Disclaimer

!!!!!!!⚠️⚠️!!!!!!!!

All the words in these sections are big WARNING that the code is using bash shell and AWS credentials to do its "magic". LLM access to shell on its own is high risk ;)

So please think about questions and experiments!

You were warned.

!!!!!!!⚠️⚠️!!!!!!!!

The code provided in this article is offered as-is, without any warranties or guarantees of any kind.

The author does not assume any responsibility for any errors, omissions, or damages that may arise from the use or misuse of the code. The user assumes all risks associated with the implementation and usage of the code.

The author shall not be held liable for any direct, indirect, incidental, special, or consequential damages that may result from the use or inability to use the code. This includes, but is not limited to, loss of data, loss of profits, business interruption, or any other commercial damages or losses.

The code is provided for educational and informational purposes only and should not be used in production environments without proper testing, validation, and security considerations.

The user is solely responsible for ensuring the code's suitability, security, and compliance with applicable laws and regulations.

The author does not endorse or recommend any specific use of the code and does not guarantee its compatibility with any particular software, hardware, or operating system. The user assumes full responsibility for any modifications made to the code and any consequences that may arise from such modifications.

By using the code, the user acknowledges and agrees to these terms and conditions. If the user does not agree with these terms, they should refrain from using or implementing the code.

!!!!!!!⚠️⚠️!!!!!!!!

## Problem

One of the common challenges AWS newcomers face is misconfigured security group rules, which can lead to denied access to resources like virtual machines (VMs) or web servers running on Elastic Compute Cloud (EC2) instances. This issue often arises due to the "deny all" principle by default in AWS, where no inbound or outbound traffic is allowed unless explicitly permitted through security group rules.

In this use case, we created a virtual cloud network (VPC) with a misconfigured firewall (security group) that prevented access to a web server running on an EC2 instance on port 80 (HTTP). This scenario is a typical newbie mistake and highlights the importance of properly configuring security group rules to allow intended traffic.

To address this issue, we leveraged the power of an AI assistant specifically designed for VPC troubleshooting. This AI agent was equipped with access to a bash shell and the AWS Command Line Interface (CLI) toolkit, enabling it to interact with AWS resources and perform necessary actions.

The AI assistant first analyzed the current configuration of the VPC, including the security groups associated with the EC2 instance hosting the web server. By identifying the missing inbound rule for HTTP traffic on port 80, the AI agent was able to add the required rule to the appropriate security group, allowing inbound traffic from any IP address (0.0.0.0/0) on port 80.

Once the security group rule was updated, the AI assistant verified that the web server could be accessed using the public IP address of the EC2 instance.

If access was still denied, the AI agent had the capability to further investigate potential issues, such as the web server configuration on the instance, ensuring the web server software was running, and verifying that the instance was in a public subnet with an Internet Gateway attached to the VPC's route table.

This use case highlights the power of an AI assistant specifically designed for VPC troubleshooting. By leveraging its knowledge and capabilities, the AI agent was able to quickly identify and resolve a common misconfiguration issue, saving valuable time and effort for AWS users, especially those new to the platform.

## Implementation

CloudFormation template to simulate VPC misconfiguration:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: VPC with EC2 Instance and Security Group
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      InstanceTenancy: default
      Tags:
        - Key: Name
          Value: MyVPC
  InternetGateway:
    Type: AWS::EC2::InternetGateway
  VPCGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.0.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select
        - 0
        - !GetAZs ''
      Tags:
        - Key: Name
          Value: Public Subnet
  RouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: Public Route Table
  Route:
    Type: AWS::EC2::Route
    DependsOn: VPCGatewayAttachment
    Properties:
      RouteTableId: !Ref RouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref RouteTable
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: SSH Access
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0cff7528ff583bf9a
      InstanceType: t2.micro
      KeyName: test-vpc
      NetworkInterfaces:
        - AssociatePublicIpAddress: 'true'
          DeviceIndex: '0'
          GroupSet:
            - !Ref SecurityGroup
          SubnetId: !Ref PublicSubnet
      UserData: !Base64 |
        #!/bin/bash
        yum update -y
        yum install -y httpd
        systemctl start httpd
        systemctl enable httpd
      Tags:
        - Key: Name
          Value: DummyHTTPServer
Outputs:
  VPC:
    Description: A reference to the created VPC
    Value: !Ref VPC
    Export:
      Name: !Sub ${AWS::StackName}-VPCID
  PublicSubnet:
    Description: A reference to the public Subnet
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub ${AWS::StackName}-SubnetID
  EC2Instance:
    Description: A reference to the EC2 instance
    Value: !Ref EC2Instance
    Export:
      Name: !Sub ${AWS::StackName}-EC2InstanceID
```

This CloudFormation template creates the following resources:

* A VPC with DNS settings and a custom tag.
    
* An Internet Gateway is attached to the VPC.
    
* A public subnet with public IP auto-assignment enabled.
    
* A route table associated with the public subnet, including a route to the Internet Gateway.
    
* A security group allowing SSH access from any IP address.
    
* An EC2 instance in the public subnet, running an Apache HTTP server.
    

## AI Agent

Here is a code for AI Agent build using the [LangChain](https://langchain.com/) framework.

```python
# Notebook to build AI Agent to fix VPC issues

# Access to Claude-3 Sonnet LLM using aws sso login to get credentials
from langchain_community.chat_models import BedrockChat
import boto3
boto3_bedrock  = boto3.client("bedrock-runtime", 'us-east-1')

model = BedrockChat(
    model_id="anthropic.claude-3-sonnet-20240229-v1:0",
    model_kwargs={
        "temperature": 0,
        "top_k": 250,
        "top_p": 1,
    },
    client=boto3_bedrock,
)

# configure react agent
from langchain.agents import AgentExecutor, create_react_agent
from langchain_community.tools import ShellTool
from langchain_core.prompts import PromptTemplate

# Initialize the language model
llm = model

# Initialize the tools
# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!! WARNING RAW SHELL EXPOSED !!!!!!!!!!!!!!!!!!!!!!!
tools = [ShellTool()]
# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!! WARNING RAW SHELL EXPOSED !!!!!!!!!!!!!!!!!!!!!!!
# !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

# Initialize prompt
template = """
You are an AWS VPC troubleshooting assistant, and your task is to identify and resolve any misconfigurations in an AWS Virtual Private Cloud (VPC) and its associated resources.

You have access to the bash shell, which allows you to run AWS CLI commands to retrieve information about AWS resources, make changes, and perform various operations.
List of tools:
{tools}
{tool_names}

First, gather information about the existing VPC setup by running the following AWS CLI commands:

1. `aws ec2 describe-vpcs`: This command will list all available VPCs in the current region, along with their CIDR blocks, VPC IDs, and other details. Review the output to ensure the VPC you want to troubleshoot exists and has the correct CIDR block.

2. `aws ec2 describe-subnets --filters Name=vpc-id,Values=<vpc-id>`: Replace `<vpc-id>` with the ID of the VPC you want to troubleshoot. This command will list all subnets associated with the specified VPC, their Availability Zones, and CIDR blocks. Verify that the subnets have the correct CIDR blocks and are associated with the intended Availability Zones.

3. `aws ec2 describe-route-tables --filters Name=vpc-id,Values=<vpc-id>`: This command will list all route tables associated with the specified VPC. Review the routes in each route table to ensure they are configured correctly and match your expectations. For example, check if there are routes to an Internet Gateway for public subnets, and routes to a NAT Gateway or VPN for private subnets.

4. `aws ec2 describe-security-groups --filters Name=vpc-id,Values=<vpc-id>`: This command will list all security groups associated with the specified VPC. Review the inbound and outbound rules for each security group to ensure they are configured correctly and allow or deny the intended traffic.

5. `aws ec2 describe-network-acls --filters Name=vpc-id,Values=<vpc-id>`: This command will list all network ACLs associated with the specified VPC. Review the inbound and outbound rules for each network ACL to ensure they are configured correctly and allow or deny the intended traffic.

After gathering this information, analyze the output and identify any potential misconfigurations or deviations from the expected setup. If you find any issues, provide step-by-step instructions on how to fix them using the appropriate AWS CLI commands.

Remember to double-check your commands before executing them, as some commands may make changes to the AWS environment.

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {input}
Thought:{agent_scratchpad}
"""
prompt = PromptTemplate.from_template(template)

# Initialize the agent
agent = create_react_agent(llm, tools, prompt)

# Run the agent
agent_executor = AgentExecutor(
    agent=agent, tools=tools, verbose=True, handle_parsing_errors=True
)

agent_executor.invoke({"input": "I cannot access web server at ec2 instance ID i-xxxxxx"})
```

Here's a section about the AI Agent based on the provided code:

The provided code demonstrates how to build an AI Agent for troubleshooting AWS Virtual Private Cloud (VPC) issues using the AWS Bedrock runtime and the Claude-3 Sonnet language model from Anthropic. This agent leverages the power of large language models (LLMs) and the AWS Command Line Interface (CLI) to identify and resolve misconfigurations in VPC setups.

### Setting up the Environment

The code begins by importing the necessary libraries and establishing a connection to the AWS Bedrock runtime using the `boto3` library. The `BedrockChat` model from the `langchain_community` package is used to interface with the Claude-3 Sonnet language model.

```python
from langchain_community.chat_models import BedrockChat
import boto3
boto3_bedrock = boto3.client("bedrock-runtime", 'us-east-1')

model = BedrockChat(
    model_id="anthropic.claude-3-sonnet-20240229-v1:0",
    model_kwargs={
        "temperature": 0,
        "top_k": 250,
        "top_p": 1,
    },
    client=boto3_bedrock,
)
```

### Configuring the Agent

The code then configures the AI agent using the `create_react_agent` function from the `langchain.agents` module. The agent is equipped with a `ShellTool` from the `langchain_community.tools` module, which allows it to execute shell commands, including AWS CLI commands.

```python
from langchain.agents import AgentExecutor, create_react_agent
from langchain_community.tools import ShellTool
from langchain_core.prompts import PromptTemplate

llm = model
tools = [ShellTool()]
```

### Defining the Prompt

A prompt template guides the agent's behaviour and provides instructions and context for the troubleshooting task. The template includes a list of AWS CLI commands that the agent can execute to gather information about the VPC setup, such as describing VPCs, subnets, route tables, security groups, and network ACLs.

```python
template = """
# ... (prompt template omitted for brevity)
"""
prompt = PromptTemplate.from_template(template)
```

### Running the Agent

Finally, the code initializes the agent using the `create_react_agent` function and runs it using the `AgentExecutor`. The agent can be invoked with a specific input question or problem statement related to VPC troubleshooting.

```python
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent, tools=tools, verbose=True, handle_parsing_errors=True
)

agent_executor.invoke({"input": "I cannot access web server at ec2 instance ID i-xxxxxx"})
```

When invoked, the agent will follow the instructions in the prompt, gathering information about the VPC setup using the AWS CLI commands. It will analyze the output, identify potential misconfigurations, and provide step-by-step instructions on how to fix any issues using the appropriate AWS CLI commands.

This AI Agent can be a valuable tool for AWS administrators and developers who are responsible for managing and troubleshooting VPC setups. By leveraging the power of large language models and the AWS CLI, the agent can quickly identify and resolve complex VPC issues, streamlining the troubleshooting process and improving overall operational efficiency.

## The result is a new one

This time I removed route to Internet Gateway in public subnet.

> **Question**: I cannot access web server at ec2 instance ID i-xxxxxx.
> 
> **Answer**: To allow internet access to the web server on the EC2 instance (i-xxxxxx), follow these steps:
> 
> 1\. Create an Internet Gateway: \`\`\` aws ec2 create-internet-gateway \`\`\`
> 
> 2\. Attach the Internet Gateway to the VPC: \`\`\` aws ec2 attach-internet-gateway --vpc-id vpc-03xxxxx --internet-gateway-id &lt;internet-gateway-id&gt; \`\`\` Replace `<internet-gateway-id>` with the ID of the Internet Gateway created in step 1.
> 
> 3\. Find the route table associated with the public subnet (subnet-03xxxxxx): \`\`\` aws ec2 describe-route-tables --filters Name=association.subnet-id,Values=subnet-03xxxxxx --query 'RouteTables\[\*\].RouteTableId' --output text \`\`\`
> 
> 4\. Create a route in the route table to the Internet Gateway: \`\`\` aws ec2 create-route --route-table-id &lt;route-table-id&gt; --destination-cidr-block 0.0.0.0/0 --gateway-id &lt;internet-gateway-id&gt; \`\`\` Replace `<route-table-id>` with the ID of the route table associated with the public subnet, and `<internet-gateway-id>` with the ID of the Internet Gateway created in step 1.
> 
> After completing these steps, instances launched in the public subnet (subnet-03xxxxxx) should be able to access the internet and be accessible from the internet. You should now be able to access the web server on the EC2 instance (i-xxxx) using its public IP address (xxx.xxx.xxx.xxx).

## Conclusion

In conclusion, leveraging Generative AI and AWS CLI to troubleshoot AWS VPC issues offers a groundbreaking approach to network management. This methodology streamlines the process of identifying and resolving misconfigurations and significantly reduces the time and effort required for these tasks. By employing a simple React AI Agent integrated with the Claude-3 Sonnet model, we demonstrated a practical application of AI in automating the process of fixing common security group misconfigurations, thereby enhancing access to EC2 instances. This exploration underscores the potential of AI in simplifying cloud networking challenges, making it an invaluable tool for AWS users seeking to optimize their networking configurations efficiently.