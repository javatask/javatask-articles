---
title: "Platform engineering. Introduction"
datePublished: Wed Jun 14 2023 08:06:58 GMT+0000 (Coordinated Universal Time)
cuid: clivfi40u000g0amr0bo9e8ph
slug: platform-engineering-introduction
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1686728737636/d457f36b-249f-4800-936a-94b980069d56.jpeg
tags: architecture, cloud-native, platform-engineering, sidecar

---

This blog post aimed to make an introduction to Platform engineering, who needs it and how the Platform team is different from other dev teams. This blog post is based on the [Team topologies book](https://teamtopologies.com/) and its authors, [Matthew Skelton](https://teamtopologies.com/people) and [Manuel Pais](https://teamtopologies.com/people)

# Team types

Platform engineering is a rapidly evolving field that aims to provide the best tools and practices for developing, deploying and operating software applications. Platform engineers are responsible for designing, building and maintaining the platforms that enable developers to focus on their core business logic and deliver value to customers faster and more reliably.

One of the key concepts in platform engineering is team topologies, which organise teams and their interactions based on the type and frequency of communication they need. Team topologies define four fundamental team types: stream-aligned teams, platform teams, enabling teams and complicated-subsystem teams. Each team type has a different purpose and mode of collaboration, and they can be combined to form different organizational structures. Henny Portman did excellent visualization of team types [in his blog post](https://hennyportman.wordpress.com/2020/05/25/review-team-topologies/) (Figure 1).

**Definition**  
*Platform team*: a team that works on the underlying platform supporting stream-aligned teams in delivery. The platform simplifies otherwise complex technology and reduces the cognitive load for teams that use it.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1686724666790/57e58a8a-ca88-49bc-9044-888b31cbf000.jpeg align="center")

Figure 1. Team types, according to Team toplogies

# Cloud-native aspect

Another important aspect of platform engineering is cloud native computing, an approach to building and running applications that exploits the advantages of the cloud computing model. Cloud-native applications are designed to be scalable, resilient, observable and portable across different environments. The Cloud Native Computing Foundation (CNCF) is an open-source foundation that hosts and supports many technologies and projects that enable cloud-native computing, such as [Kubernetes](https://kubernetes.io/), [Prometheus](https://prometheus.io/), [Envoy](https://www.envoyproxy.io/), [Istio](https://istio.io/), [Open Policy Agent](https://www.openpolicyagent.org/), [OpenTelemetry](https://opentelemetry.io/), [Jaeger](https://www.jaegertracing.io/), [Helm](https://helm.sh/) and others.

Figure 2 visualises the complexity of the current production-ready reference architecture.

![Figure 1. Reference cloud-native architecture](https://cdn.hashnode.com/res/hashnode/image/upload/v1686726694444/07e4f0fc-8dcd-4899-8469-00d56a2420fe.png align="center")

Figure 2. Reference cloud-native architecture

Platform engineering configures and operates sidecar proxy and infrastructure services, helping stream-aligned teams focus on developing core business logic.

**Definition**.  
*Cloud-native* is the software approach of building, deploying, and managing modern applications in cloud computing environments.  
Source: [AWS](https://aws.amazon.com/what-is/cloud-native/#:~:text=The%20term%20cloud%20native%20refers,container%20orchestrators%2C%20and%20auto%20scaling.)

# Reference architecture deep dive

Figure 2 shows how to use sidecar proxies and platform infrastructure. A sidecar proxy is a small container that runs alongside a primary container. The sidecar proxy can add functionality to the main container, such as authentication, authorization, monitoring, etc.

The platform is a pre-configured Kubernetes resource that can quickly and easily deploy business logic. The platform includes a sidecar proxy configured to use REST API.

At the bottom of Figure 2, you can find an example of preconfigured infrastructure services integration. This part of the figure illustrates how the Platform team can implement database integration with a message broker via an outbox pattern.

The outbox pattern is a technique for implementing reliable messaging between services. It consists of two main steps: first, the service writes any messages it needs to send to a local database table (the outbox); second, a separate process (the outbox publisher) reads the messages from the outbox and sends them to the message broker.

The benefits of using the outbox pattern are:

* It ensures that messages are not lost or duplicated, even if the service or the message broker fails.
    
* It decouples the service from the message broker, making it easier to test and maintain.
    
* It avoids blocking the service while waiting for message delivery acknowledgements.
    

A platform simplifies the implementation of the outbox pattern. It provides:

* A configuration that integrates with popular databases (such as PostgreSQL, MySQL, MongoDB, etc.) allows you to easily write messages to the outbox table using a fluent API.
    
* A service that monitors the outbox tables and publishes the messages to the message broker (such as Kafka, RabbitMQ, etc.) using various strategies (such as polling, triggers, etc.).
    
* A dashboard that lets you monitor and manage the outbox tables and publishers.
    

The business logic part of a Platform is compatible with different programming languages (such as Java, Python, Node.js, etc.) and messaging formats (such as JSON, Avro, Protobuf, etc.). It also supports transaction models (such as 2PC, saga, etc.) and message ordering guarantees (such as FIFO, causal, etc.).

The following are some of the benefits of using sidecar proxies and platforms:

**Quick and easy deployment:** The platform includes pre-configured Kubernetes resources that can be used to quickly and easily deploy and operate your custom business logic.

**Flexible:** The sample platform can be used to deploy business logic on a variety of platforms, including Kubernetes, AWS, Azure and others.

**Secure:** The sidecar proxy can be used to add security features to the main container, such as authentication and authorization, without changing the business logic code.

**Scalable:** The sample platform can be scaled to meet the needs of your application.

In this blog post, we will explore some of the benefits and challenges of platform engineering, how team topologies can help optimize the flow of work and feedback, and how cloud-native technologies can enhance the performance and reliability of software applications. We will also share some best practices and resources to learn about platform engineering.