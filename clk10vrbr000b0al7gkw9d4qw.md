---
title: "IoT Edge Runtime with AWS Greengrass. Tunneling."
seoTitle: "AWS Greengrass IoT Edge Runtime Tunneling"
seoDescription: "Establish secure EC2-Rest API communication in AWS Greengrass using IoT Secure Tunneling for efficient cloud infrastructure"
datePublished: Thu Jul 13 2023 10:44:00 GMT+0000 (Coordinated Universal Time)
cuid: clk10vrbr000b0al7gkw9d4qw
slug: iot-edge-runtime-with-aws-greengrass-tunneling
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1699350940139/d3f42db5-aba7-49a4-b25f-6417388f6b7e.png
tags: aws, security, tunnel, aws-iot-core

---

# Cost warning

⚠**Warning!!**! ⚠ AWS IoT Core Secure tunnelling cost you 1.2$ for EACH opened tunnel. But, regeneration of secure tokens is free for the already opened tunnels. This tutorial will require a single tunnel, thus it will cost you 1.2$. For more information please visit [AWS IoT Core pricing page](https://aws.amazon.com/iot-device-management/pricing/).

# Introduction

In today's interconnected world, secure communication is a crucial aspect of any cloud-based system. In this article, we'll discuss a workflow that utilizes AWS IoT Secure Tunneling to facilitate a secure connection between an EC2 instance and a Rest API server running in AWS Greengrass.

# Parts

[This series consists of four parts, each part is independent of another](https://blog.javatask.dev/series/iot).

[**Introduction**. In this part, we are talking about the use case and the AWS Greengrass itself](https://blog.javatask.dev/iot-edge-runtime-with-aws-greengrass-introduction)

[**Setup**. Here I'll show how to quickly launch Greengrass as a docker image on our computer](https://blog.javatask.dev/iot-edge-runtime-with-aws-greengrass-setup)

[**Custom Component**. This part shows how to deploy a sample Java REST API server to an edge device running AWS Greengrass](https://blog.javatask.dev/iot-edge-runtime-with-aws-greengrass-build-and-deploy-custom-component)

**Secure tunneling**. This part shows how to build a tunnel from your AWS Greengrass device to the Cloud, together with the ability to send requests from the cloud to your REST API server deployed at Edge without a VPN connection.

# Compile AWS local proxy

Before we can start building the tunnel there is a need to compile **localproxy** binary [https://github.com/aws-samples/aws-iot-securetunneling-localproxy](https://github.com/aws-samples/aws-iot-securetunneling-localproxy) or as an alternative extract it from [docker images](https://gallery.ecr.aws/aws-iot-securetunneling-localproxy).

In this section, we will learn how to compile the localproxy application from the AWS IoT Secure Tunneling service using the [docker-build.sh](http://docker-build.sh) command. The localproxy application is a tool that enables bidirectional communication between devices and services through secure tunnels. The [docker-build.sh](http://docker-build.sh) command is a script that simplifies the compilation process by using a Docker container.

To compile the localproxy application, we need to follow these steps:

1. Clone the aws-iot-securetunneling-localproxy repository from GitHub: `git clone` [`https://github.com/aws-samples/aws-iot-securetunneling-localproxy.git`](https://github.com/aws-samples/aws-iot-securetunneling-localproxy.git)
    
2. Navigate to the localproxy folder: `cd aws-iot-securetunneling-localproxy`
    
3. Edit Docker file by changing line 2 and 92 from `amazonlinux:latest` to `amazonlinux:2`
    
4. Run the [docker-build.sh](http://docker-build.sh) command: [`./docker-build.sh`](http://docker-build.sh)
    
5. The compiled binary will be located in the newly created docker image. You need to run your image `aws-iot-securetunneling-localproxy:latest` and extract localproxy binaries `docker cp CONTAINER:localproxy .` and `docker cp CONTAINER:localproxytest .`
    
6. Test that your localproxy is working by running `./localproxytest`. I highly recommend using this test suite inside your docker images to check that all localproxy dependencies were met.
    

For more information on how to use the localproxy application and its options, refer to the [README](https://github.com/aws-samples/aws-iot-securetunneling-localproxy) file in the repository or the [AWS IoT Secure Tunneling documentation](https://docs.aws.amazon.com/iot/latest/developerguide/secure-tunneling.html).

# **Copying the Localproxy to AWS Greengrass**

The initial step of the workflow involves transferring a localproxy file from your local system to AWS Greengrass. The file contains the localproxy tool, which is required for creating the secure tunnel between the EC2 instance and AWS Greengrass.

> **Note**. In production, you may include **localproxy** binary into your container, by adding COPY commands to your AWS Greengrass Docker file.

We can achieve this by using the Docker `cp` command. The Docker `cp` command is used to copy files or folders between a Docker container and the local filesystem. In this context, we're copying the localproxy file from our local system to a running AWS Greengrass Docker container.

Here is the copy command:

```bash
docker cp ./localproxytest my-iot-greengrass:/
```

In this command:

* `docker cp`: The base command that you're using to copy files or directories.
    
* `./localproxytest`: This is the source path. It represents the file in your local system named `localproxytest` that is located in the current working directory (`./`).
    
* `my-iot-greengrass:/`: This is the destination path. It's copying the `localproxytest` file into the root directory (`/`) of the Docker container named `my-iot-greengrass`.
    

Ensure the AWS Greengrass container is running before you run the Docker `cp` command. Also, ensure that you have the necessary permissions to execute the `cp` command.

Now, our localproxy tool is successfully copied to AWS Greengrass, and we're ready to proceed to the next step, opening bash inside your container.

# **Bashing into AWS Greengrass container**

## **Check AWS Greengrass container**

Before you can access bash for a Docker container, you need to ensure that the container is running. Open up a terminal window and type in the following command:

```bash
docker ps
```

This command will show you a list of all currently running Docker containers. The output will contain several columns, including the Container ID, Image, Command, Created, Status, Ports, and Names.

You need to find your container named `my-iot-greengrass`

## **Access the AWS Greengrass container's bash**

Next, to access the Docker container's bash, you can use the `docker exec` command, which allows you to run commands inside a running container. The syntax for accessing bash in a container is as follows:

```bash
docker exec -it my-iot-greengrass /bin/bash
```

In this command, replace `my-iot-greengrass` is the actual ID of the Docker container that you want to access.

The `-it` flag is a combination of two separate flags:

* `-i` (or `--interactive`) keeps STDIN open, allowing you to interact with the container,
    
* `-t` (or `--tty`) allocates a pseudo-TTY, essentially emulating a terminal.
    

The `/bin/bash` part of the command tells Docker to start a Bash shell inside the container.

After running the command, if your Docker container has a bash shell installed, you will get bash access in the terminal.

Keep shell open, we will open another shell on AWS EC2 and then we will run localproxy on destination (AWS Greengrass) and source (AWS EC2) to build the tunnel.

# Running EC2 as Central Management System

This section contains steps for creating an Amazon EC2 instance via the AWS CLI (Command Line Interface) and then copying a localproxy file to the instance using SCP (Secure Copy).

This tutorial assumes you have the AWS CLI installed and configured with your AWS account. Also, you should have an SSH key pair for connecting to your EC2 instance.

1. **Creating an EC2 Instance using AWS CLI**
    
    To launch an instance, first, you need to know the ID of the AMI that you will use. For example, the ID for Amazon Linux 2 Kernel 5.10 AMI 2.0.20230628.0 x86\_64 HVM gp2 AMI in eu-central-1 region is `ami-0aea56f3589631913`. Use the `describe-images` command to find AMIs.
    
    ```bash
    aws ec2 describe-images --owners amazon
    ```
    
    Now, launch an EC2 instance using the `run-instances` command.
    
    ```bash
    aws ec2 run-instances --image-id ami-0aea56f3589631913 \
         --count 1 --instance-type t2.micro \
         --associate-public-ip-address \
         --key-name MyKeyPair
    ```
    
    Here, replace `ami-0abcdef1234567890` and `MyKeyPair` with the AMI ID and your key pair name. This will launch a `t2.micro` instance.
    
    You should see JSON output that includes the instance ID (i.e., `"InstanceId": "i-0abcdef1234567890"`).
    
2. **Obtaining Public DNS Name or IP Address**
    
    Use the `describe-instances` command with the instance ID to find its public DNS name or IP address.
    
    ```bash
    aws ec2 describe-instances --instance-ids i-0abcdef1234567890
    ```
    
    In the output, find the `"PublicDnsName"` or `"PublicIpAddress"` value.
    
3. **Copying a File to the EC2 Instance using SCP**
    
    SCP relies on SSH. When you created the instance, you should have specified an SSH key pair for the instance. You'll need the private key file (.pem) for this SSH key pair to use SCP.
    
    Use the `scp` command to copy a file to the EC2 instance.
    
    ```bash
    scp -i MyKeyPair.pem ./localproxy ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:
    ```
    
    Here, replace `MyKeyPair.pem` with your private key file and [`ec2-198-51-100-1.compute-1.amazonaws.com`](http://ec2-198-51-100-1.compute-1.amazonaws.com) with your instance's public DNS name or IP address.
    

That's it! You've created an Amazon EC2 instance using the AWS CLI and copied a local file to the instance using SCP.

**NOTE:** Make sure the security group associated with your EC2 instance allows inbound SSH traffic from your IP address. Otherwise, the SCP command may fail.

# **Creating an AWS IoT Secure Tunnel**

After establishing a bash connection to our AWS Greengrass Core, the next step is to create a secure tunnel. AWS IoT Secure Tunneling is a managed proxy meant for devices located behind secure firewalls. It provides a secure communication channel between the device and AWS services.

To create a secure tunnel, you can use the `aws iotsecuretunneling open-tunnel` command. The general form of this command is:

```bash
 aws iotsecuretunneling open-tunnel --region eu-central-1
```

Output is

```json
{
    "tunnelId": "16385d13-36c3-4a2b-b959-0ec759cfffda",
    "tunnelArn": "arn:aws:iot:eu-central-1:1111111111:tunnel/16385d13-36c3-4a2b-b959-0ec759cfffda",
    "sourceAccessToken": "AQGAAXiXnUEpE-lNeuNTMwNxtMu9ABsAAgABQQAMMzMyMDA2MTE3MTc5AAFUAANDQVQAAQAHYXdzLWxxxxx0=",
    "destinationAccessToken": "AQGAAXjrxuHKTaBd8-lM4yiKCzk9ABsAAgABQQAMMzMyMDA2MTE3MTc5AAFUAANDQVQAAQAHYXdzLWttcwBOYXJuOmF3czprbXM6ZXUtY2VudHJhbC0xOjk5NzcwNzY2ODE2MTprZXkvxxxxxxxx="
}
```

And then you can run your destination localproxy in your AWS Greengrass container bash with the token (-t) value taken from `destinationAccessToken` field.

```bash
./localproxy -r eu-central-1 -d localhost:3000 -t  "AQGAAXjrxuHKTaBd8-lxxxxx"
```

and source proxy on EC2 instance with the token (-t) value taken from `sourceAccessToken` field.

```bash
 ./localproxy -r eu-central-1 -s 3000 -t  "AQGAAXiXnUEpE-xxxx"
```

# **Curl via Secure Tunnel to your Rest API**

The last step is to execute a curl command inside the EC2 instance to reach the Rest API server running in AWS Greengrass. This request is sent via the secure tunnel we set up earlier.

```shell
curl -X GET http://localhost:8080/greeting
```

Response:

```json
{"id":1,"content":"Hello, World!"}
```

That's it! We've successfully set up secure communication from an EC2 instance to AWS Greengrass using AWS IoT Secure Tunneling.

> **Note**. Please be aware that the creation of a tunnel for 12 hours costs 1.2$ (one dollar and 20 cents). Also please keep in mind that the maximum [tunnel throughput is 100 KBps (800kbps)](https://aws.amazon.com/iot-device-management/pricing/).

# Summary

In this article, we discussed the workflow to establish secure communication between an EC2 instance and a Rest API server in AWS Greengrass via AWS IoT Secure Tunneling. The process involves creating and configuring AWS resources and running the localproxy in source and destination modes. By utilizing this workflow, we can ensure secure and efficient communication within our cloud infrastructure.

# Bonus

### **SecureTunnel Component of AWS Greengrass (additional)**

SecureTunnel Component of AWS Greengrass is a feature that enables secure and bidirectional communication between devices and services in the cloud. It uses MQTT protocol and TLS encryption to establish a tunnel that can work with firewalls and proxies. SecureTunnel Component of AWS Greengrass can be used for remote device management, data collection, and troubleshooting.

I will use AWS CLI to deploy the SecureTunnel component to AWS Greengrass using AWS CLI.

You need to create a deployment for your Greengrass group. The deployment will contain the AWS IoT Secure Tunneling connector.

First, we need a JSON document which will describe the components to be deployed. Save this content into `securetunnel.json`:

```json
{
  "components": {
    "aws.greengrass.SecureTunneling": {
      "componentVersion": "1.0.16"
    },
    "aws.greengrass.Nucleus": {
      "componentVersion": "2.11.0"
    }
  }
}
```

The next step is to get [ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html) of your AWS Greengrass device, and run the command:

```bash
aws iot list-things
```

You should get JSON response with a single Thing, sample:

```json
{
    "things": [
        {
            "thingName": "GreengrassV2IotThing_8904a441-3a2d-4c1e-9b2b-33b6c7c9a2b2",
            "thingArn": "arn:aws:iot:eu-central-1:111111111:thing/GreengrassV2IotThing_8904a441-3a2d-4c1e-9b2b-33b6c7c9a2b2",
            "attributes": {},
            "version": 1
        }
    ]
}
```

Copy `thingArn` to the next command:

```bash
aws greengrassv2 create-deployment \
    --target-arn "arn:aws:iot:REGION:ACCOUNT_ID:thing/GREENGRASS_THING_NAME" \
    --deployment-name "SecureTunnelDeployment" \
    --cli-input-json file://securetunnel.json
```

Wait until the deployment is in COMPLETED state via `aws greengrassv2 get-deployment --deployment-id 31fxxxxxxx` command.

Using AWS IoT Core Console you can SSH into your Thing IF you know the user and password or user and private key.