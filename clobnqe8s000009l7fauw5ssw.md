---
title: "Deploying a Single Page Application (SPA) on AWS: A Beginner's Guide. Part 2. Secure Static Hosting"
seoTitle: "SPA Deployment on AWS: Beginner's Guide Pt. 2"
seoDescription: "Deploy secure static hosting for Single Page Applications on AWS with CloudFormation, Amazon S3, and CloudFront in this beginner's guide"
datePublished: Sun Oct 29 2023 16:00:12 GMT+0000 (Coordinated Universal Time)
cuid: clobnqe8s000009l7fauw5ssw
slug: deploying-a-single-page-application-spa-on-aws-a-beginners-guide-part-2-secure-static-hosting
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1697627725150/0204fbcc-cd8a-4c39-ba3a-f9c3da40f88f.png
tags: hosting, aws, spa, frontend-development, s3

---

## Introduction

This section outlines the essential steps to deploy the necessary infrastructure for hosting your app's Frontend static assets securely.

# Source code repository

[javatask/aws-spa-beginners-guide: Code for the blog series on porting single page application to AWS (](https://github.com/javatask/aws-spa-beginners-guide)[github.com](http://github.com)[)](https://github.com/javatask/aws-spa-beginners-guide)

## Use Case

![Static hosting with Cloudfront and S3](https://cdn.hashnode.com/res/hashnode/image/upload/v1690547418566/72ae729d-1b89-4e64-ad79-ffed8a162ba3.png align="left")

Figure 1: Secure Static Hosting with Cloudfront and S3

As depicted in Figure 1, we will deploy two core resources: a Cloudfront distribution and an S3 bucket.

To accomplish this, we will leverage the concept of Infrastructure as Code (IaC). IaC empowers you to define your infrastructure as code, allowing you to create resources effortlessly. In AWS, the primary service responsible for IaC is CloudFormation (CFN). In this guide, we'll utilize CFN to create the Cloudfront distribution and S3 bucket needed for secure static hosting.

Let's dive into the practical implementation.

## Infrastructure as Code Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: Static contents distribution using S3 and CloudFront.
Resources:
  # S3 bucket contains static contents
  AssetsBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      BucketName: !Sub '${AWS::StackName}-${AWS::AccountId}'
      BucketEncryption:
        ServerSideEncryptionConfiguration: 
          - ServerSideEncryptionByDefault: 
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration: 
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      LoggingConfiguration:
        DestinationBucketName: !Ref AccessLogBucket
        LogFilePrefix: origin/

  # S3 bucket policy to allow access from CloudFront OAI
  AssetsBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref AssetsBucket
      PolicyDocument:
        Statement:
        - Action: s3:GetObject
          Effect: Allow
          Resource: !Sub ${AssetsBucket.Arn}/*
          Principal:
            Service: cloudfront.amazonaws.com
          Condition:
            StringEquals:
              AWS:SourceArn: !Sub arn:aws:cloudfront::${AWS::AccountId}:distribution/${AssetsDistribution}
        - Effect: Deny
          Principal: '*'
          Action: 's3:*'
          Resource: 
            - !Sub ${AssetsBucket.Arn}/*
            - !GetAtt AssetsBucket.Arn
          Condition:
            Bool: 
              aws:SecureTransport: false

  AccessLogBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    UpdateReplacePolicy: Retain
    Properties:
      BucketName: !Sub '${AWS::StackName}-${AWS::AccountId}-logs'
      BucketEncryption:
        ServerSideEncryptionConfiguration: 
          - ServerSideEncryptionByDefault: 
              SSEAlgorithm: AES256
      LifecycleConfiguration:
        Rules:
          - Id: Retain2yrs
            Status: Enabled
            ExpirationInDays: 730
            Transitions:
              - StorageClass: STANDARD_IA
                TransitionInDays: 30
      PublicAccessBlockConfiguration: 
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true

  AccessLogBucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref AccessLogBucket
      PolicyDocument:
        Statement:
          - Effect: Deny
            Principal: '*'
            Action: 's3:*'
            Resource: 
              - !Sub ${AccessLogBucket.Arn}/*
              - !GetAtt AccessLogBucket.Arn
            Condition:
              Bool: 
                aws:SecureTransport: false

  # CloudFront Distribution for contents delivery
  AssetsDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Origins:
        - Id: S3Origin
          DomainName: !GetAtt AssetsBucket.DomainName
          S3OriginConfig:
            OriginAccessIdentity: ''
          OriginAccessControlId: !GetAtt CloudFrontOriginAccessControl.Id
        Enabled: true
        DefaultRootObject: index.html
        Comment: !Sub ${AWS::StackName} distribution
        DefaultCacheBehavior:
          CachePolicyId: '4135ea2d-6df8-44a3-9df3-4b5a84be39ad' # Cache disabled
          TargetOriginId: S3Origin
          ViewerProtocolPolicy: redirect-to-https
        HttpVersion: http2
        ViewerCertificate:
          CloudFrontDefaultCertificate: true
        IPV6Enabled: false

  CloudFrontOriginAccessControl:
    Type: AWS::CloudFront::OriginAccessControl
    Properties: 
      OriginAccessControlConfig:
        Description: Default Origin Access Control
        Name: !Ref AWS::StackName
        OriginAccessControlOriginType: s3
        SigningBehavior: always
        SigningProtocol: sigv4

Outputs:
  BucketWithStaticAssets:
    Value: !Ref AssetsBucket
  URL:
     Value: !Join [ "", [ "https://", !GetAtt AssetsDistribution.DomainName ]]
```

**Code Listing 1:** AWS CloudFormation template for deploying resources for secure static hosting.

To commence the deployment of the CFN template (as seen in Code Listing 1), execute the following command:

```bash
aws cloudformation deploy --stack-name your-uniq-stack-name --template-file frontend.yaml
```

> **Note**: Ensure the use of a lowercase stack name, as it is employed to generate a unique S3 bucket name. S3 only allows lowercase characters.

To monitor the status of your CFN stack, utilize the following command:

```bash
aws cloudformation describe-stacks --stack-name your-uniq-stack-name
```

You should wait for the stack status to reach `CREATE_COMPLETE`. After the creation, retrieve

the stack outputs with the command:

```bash
aws cloudformation describe-stacks --stack-name your-uniq-stack-name --query 'Stacks[0].Outputs'
```

You should receive the following output:

```yaml
[
    {
        "OutputKey": "BucketWithStaticAssets",
        "OutputValue": "your-uniq-stack-name-11111111"
    },
    {
        "OutputKey": "URL",
        "OutputValue": "https://xxxxxxxx.cloudfront.net"
    }
]
```

This output grants you the ability to upload static assets to an S3 bucket and access them via the `https://xxxxxxxx.cloudfront.net` URL.

> ⚠️ **Warning**: DNS propagation typically takes 4-12 hours. Attempting to access the Cloudfront URL prematurely may result in a "Permission denied" exception, displaying the raw S3 URL. This indicates that DNS is not yet ready, so please be patient.

Before delving into a detailed explanation of each resource in the CFN template, upload the `index.html` file to the newly created S3 bucket:

```bash
aws s3 cp index.html s3://your-uniq-stack-name-11111111
```

To list the files in your S3 bucket, use:

```bash
aws s3 ls s3://your-uniq-stack-name-111111111
```

Wait patiently, and you'll witness a simple HTML page on your Cloudfront URL.

Now, let's dive into the CFN template.

## Demystifying the CloudFormation Template

This CloudFormation template facilitates the distribution of static content using Amazon S3 and CloudFront. Here's an overview of its key components:

* **Parameters**: This section is absent
    
* **Mappings**: This section is absent
    
* **Resources**:
    
    * `AssetsBucket`: This AWS S3 bucket holds the static content. Its name combines the stack name and the AWS account ID. The bucket is configured with server-side AES256 encryption and is protected against public access. S3 bucket logging is enabled, with logs stored in the `AccessLogBucket`.
        
    * `AssetsBucketPolicy`: This is a bucket policy for the S3 bucket, allowing CloudFront to retrieve objects from the bucket (specifically, the GetObject action). It also includes a statement to deny all S3 actions if the request is not made over a secure (SSL/TLS) connection.
        
    * `AccessLogBucket`: Another S3 bucket is created to store access logs from the `AssetsBucket`. The bucket name includes the stack name, AWS account ID, and a '-logs' suffix. This bucket is configured with server-side AES256 encryption and is protected against public access. A lifecycle rule is set to move objects to the STANDARD\_IA storage class after 30 days and delete them after 2 years.
        
    * `AccessLogBucketPolicy`: This bucket policy for the log bucket denies all S3 actions if the request is not made over a secure (SSL/TLS) connection.
        
    * `AssetsDistribution`: This CloudFront distribution is created for content delivery. It uses the S3 bucket as its origin and implements the cache policy. The default root object is set to `index.html`, and the viewer protocol policy redirects all HTTP traffic to HTTPS. It also uses the HTTP/2 protocol and employs the default CloudFront certificate.
        
    * `CloudFrontOriginAccessControl`: This resource configures CloudFront origin access control for S3, ensuring that signing behavior is always enforced with sigv4 signing protocol.
        
* **Outputs**: The template provides two outputs: the S3 bucket name for storing static assets and the URL of the CloudFront distribution.
    

This CloudFormation template allows you to rapidly set up an S3 bucket and a CloudFront distribution, ensuring secure and efficient delivery of static assets.

## In Conclusion

In this article, we've presented a comprehensive guide on deploying a secure static hosting infrastructure using AWS CloudFormation, Amazon S3, and CloudFront. We've covered the creation of an S3 bucket for storing static assets, the setup of a CloudFront distribution for content delivery, and the configuration of cache policies and security settings. The provided CloudFormation template can be easily customized to suit your specific requirements, enabling you to swiftly establish a secure and efficient static hosting solution for your app's Frontend.

Stay tuned for the next instalment - "Building Rest API"