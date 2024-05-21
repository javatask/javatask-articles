---
title: "Introduction to AWS SiteWise: Part 1. An IIoT OPC UA End-to-End Guide"
seoTitle: "AWS SiteWise: IIoT OPC UA Guide"
seoDescription: "An introductory guide to AWS IoT SiteWise for monitoring network devices using OPC UA in IIoT solutions"
datePublished: Tue May 21 2024 08:40:57 GMT+0000 (Coordinated Universal Time)
cuid: clwg5b5d0001f0al1dtk15yd8
slug: introduction-to-aws-sitewise-part-1-an-iiot-opc-ua-end-to-end-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1716277960787/462ea5df-bbfc-4269-a678-adf87794eb15.webp
tags: aws, architecture, iot, guide, iiot, opcua, sitewise

---

## **Introduction: AWS IoT SiteWise for Network Device Monitoring**

Welcome to this new blog series, where we explore the capabilities of A[WS IoT SiteWise](https://aws.amazon.com/de/iot-sitewise/) and unveil the "magic" it offers for building end-to-end (E2E) Industrial Internet of Thing (IIoT) solutions focused on monitoring the physical health of network devices. While AWS IoT Core is also a critical component of IoT strategy, this series will concentrate exclusively on SiteWise, leaving IoT Core to be covered in depth in a future series.

In this initial installment, we'll outline our project's broad objectives, providing a roadmap of what we aim to achieve and the technology that will help us achieve it.

The genesis of this series stems from a practical need within my fictional role as a network administrator. Tasked with developing a system to monitor crucial aspects of our operational technology (OT) network devices—such as power supply status, temperature levels, and port functionality—I discovered that these devices are equipped with [OPC UA](https://en.wikipedia.org/wiki/OPC_Unified_Architecture) protocol capabilities. OPC UA will be instrumental in enabling us to track and manage the health of our network equipment effectively.

Join me as we delve into AWS IoT SiteWise, leveraging its powerful features to create a robust monitoring system that ensures our network devices operate at their peak and alert us to potential issues before they impact our operations.

## **The Big Picture: Integrating OT Network with AWS SiteWise**

The journey to fully integrate the network device into our network management strategy involves several key components, each serving a distinct purpose in a larger ecosystem. This approach enhances the monitoring and management of network devices and ensures data integrity and security from the edge to the cloud (Figure 1).

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716278558286/402c3900-0703-43e8-a51b-04001232d0f6.png align="center")

> Note. Diagrams are avaliable as a code in github repo for this series - [aws-sitewise-iiot-e2e/diagrams/big\_](https://github.com/javatask/aws-sitewise-iiot-e2e/blob/main/diagrams/big_picture.py)[picture.py](http://picture.py)[at main · javatask/aws-sitewise-iiot-e2e (](https://github.com/javatask/aws-sitewise-iiot-e2e/blob/main/diagrams/big_picture.py)[github.com](http://github.com)[)](https://github.com/javatask/aws-sitewise-iiot-e2e/blob/main/diagrams/big_picture.py)

#### [**On-Premise Operations**](https://github.com/javatask/aws-sitewise-iiot-e2e/blob/main/diagrams/big_picture.py)

[Our system begins in the operational technology (OT) environment, where ph](https://github.com/javatask/aws-sitewise-iiot-e2e/blob/main/diagrams/big_picture.py)ysical devices grouped under "OT Network 1" are connected and managed. These devices, crucial to our daily operations, are first connected through a Hirschmann Industrial Operating System (HIOS 1) switch, which acts as the data source.

#### **Ensuring Security and Integrity**

Data from the OT network is then securely transmitted through "Firewall 1", ensuring that only authorized data flows into the IT network. This firewall acts as a critical barrier, guarding against potential threats and unauthorized access, thus maintaining the integrity of our operational data.

#### **Bridging OT and IT**

Once through the firewall, data moves into the IT network, where it is pre-processed and aggregated. This step is crucial for reducing noise and ensuring that only meaningful data is forwarded to the cloud for further analysis.

#### **Edge Computing**

At this stage, "Edge Compute" plays a vital role. It involves processing data locally, near the source of data generation, which minimizes latency and reduces the load on cloud services. Edge computing is crucial for real-time applications where immediate data processing is required to make timely decisions. Our OPC UA Client will be deployed to listen for the data from the network devices.

#### **AWS Cloud Integration**

Post-edge processing, data is sent to AWS Cloud, specifically to AWS SiteWise. Here, it's modelled, stored, and analyzed. AWS SiteWise provides a robust framework for creating comprehensive asset models that reflect our network's hierarchy, facilitating advanced monitoring and analytics.

#### **Visualization and Control**

The final component in our IIoT architecture is the "Dashboard," which is accessible by network administrators. This dashboard provides a centralized, intuitive interface for monitoring and managing the network's health and performance. It offers real-time insights and analytics, enabling quick decision-making and effective operational management.

Through AWS SiteWise, we manage and automate data flows from the ground up, ensuring every layer of our network is optimized for performance, reliability, and security. This strategic integration empowers our network administrators with the tools they need to excel in today's fast-paced digital environment.

## **What is AWS IoT SiteWise?**

AWS IoT SiteWise is a managed service from Amazon Web Services designed to simplify collecting, organizing, and analyzing industrial equipment data. This service is tailored to help companies capture data consistently across devices, understand their operational performance, and identify efficiencies within industrial operations.

At its core, SiteWise enables users to define and model their industrial operations as hierarchical assets, making organising and managing data from complex environments easier. This modeling capability allows for creating virtual representations of physical operations, including devices, machinery, and processes, enabling structured data ingestion and storage.

SiteWise also automates the process of data collection and ingestion from various sources like OPC-UA (Open Platform Communications Unified Architecture) and other industrial protocols, removing the need for custom-built data collection solutions. Once data is in SiteWise, the service provides built-in capabilities for monitoring equipment across facilities, quickly identifying issues with real-time alerts, and executing deeper analysis using AWS's powerful analytics tools.

SiteWise includes an integrated dashboard feature known as SiteWise Monitor for visualisation and operational insight. This tool allows industrial engineers and operators to create customizable, interactive dashboards to view and manage operational data in real-time. These dashboards can help identify bottlenecks, optimize processes, and improve overall operational health without the need for deep technical expertise.

AWS IoT SiteWise offers a comprehensive, scalable, and secure platform for industrial IoT applications (IIoT). It aids organizations in their journey toward digital transformation by leveraging detailed insights from their equipment data.

## **Series Logical Blocks: Mapping Our Journey**

This series will break down our journey into several key stages, each designed to systematically build and enhance our network monitoring system using AWS IoT SiteWise. Here's how we'll proceed:

1. **Configuring Network Devices for OPC UA:** Network devices need to be set up initially to enable OPC UA and configure the necessary credentials. While I won't cover this process in detail, I recommend using the [node-opcua/opcua-commander](https://github.com/node-opcua/opcua-commander), a client that simplifies testing OPC UA connectivity.
    
2. **\[Part 2\] Configuring Edge Compute/Gateway:** Setting up our edge computing infrastructure or gateway is crucial for piping data from OPC UA Servers to the cloud.
    
3. **\[Part 3\] Creating SiteWise Asset Models:** We'll define and configure the asset models within AWS SiteWise to represent our physical devices and their data hierarchically.
    
4. **\[Part 4\] Binding Data Streams to SiteWise Assets:** This step involves linking real-time data streams to our previously configured asset models, ensuring data flows correctly through our system.
    
5. **\[Part 5\] Developing a SiteWise Monitor Portal Dashboard:** Finally, we will build a comprehensive dashboard within the SiteWise Monitor Portal. This dashboard will serve as the central interface for visualizing and managing the health and status of our network devices.
    

This series aims to guide you through all the necessary steps, from groundwork to full implementation, ensuring a deep understanding of each setup phase.

## **Conclusion: Laying the Foundation for IIoT Success with AWS SiteWise**

As we conclude the first part of our journey into leveraging AWS IoT SiteWise for monitoring network devices, we have outlined the roadmap and set the stage for a transformative monitoring solution. By breaking down the journey into manageable steps, we have prepared the groundwork for a robust end-to-end monitoring system that will enhance operational efficiency and ensure the health of network devices.

In the upcoming parts of this series, we will dive deeper into each of the steps, starting with setting up edge computing capabilities and then creating and configuring SiteWise Asset Models. Each part will build upon the last, gradually piecing together a comprehensive solution that not only effectively captures and analyzes data but also provides actionable insights through a well-designed dashboard.

Stay tuned for the next instalment, where we will explore the configuration of Edge Compute/Gateway, a critical component in reducing latency and enabling real-time data processing at the network's edge. Your engagement and feedback are valuable as we navigate this IIoT journey, ensuring each step adds tangible value to your network management strategy.

Thank you for joining me for this introductory part. I look forward to continuing this journey with you.