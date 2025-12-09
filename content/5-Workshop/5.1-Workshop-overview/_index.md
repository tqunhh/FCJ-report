---
title: "5.1 Overview of the Workshop"
weight: 1
---


## **Introduction**

This workshop provides an overview of the **Travel Journal Web Application** built during the project.  
The main goal is to help learners understand how to design and deploy a modern web system using a **serverless architecture on AWS**, integrating services such as **AWS Rekognition**, **AWS Location Service**, **DynamoDB**, and **Amazon S3** to create a scalable, cost-optimized, and practical product.

The platform allows users to record their travel experiences by uploading photos, tagging locations, creating posts, and sharing personal journeys.  
When a photo is uploaded, **Amazon Rekognition** analyzes its content and automatically assigns labels.  
Metadata of each post is saved to **DynamoDB**, images are stored in **S3**, and static content is delivered through **Amazon CloudFront** to ensure fast performance globally.

---

## **Key Feature – Location Mapping**

A unique highlight of the system is the ability to visualize user travel routes using **AWS Location Service**, displaying the geographic points of each post on an interactive map.

---


## **Serverless Backend Architecture**

The backend is fully built on a serverless stack:

- **AWS Lambda** – handles business logic  
- **API Gateway** – exposes REST APIs  
- **DynamoDB** – stores post and user metadata  
- **S3 + SQS + Lambda pipeline** – processes uploaded images  
- **Rekognition** – automatic labeling  
- **CloudFront** – high-performance content delivery  

This architecture eliminates server management and automatically scales based on workload.

---


## **Frontend Implementation**

The frontend is a React-based SPA deployed on either:
 
- **AWS CloudFront**

ensuring easy deployment and fast global access.

---

## **Workshop Goal**

By the end of this workshop, learners will understand:

- How to integrate multiple AWS services to build a complete web system  
- How serverless architecture works  
- How to process media data with AI Rekognition  
- How to deploy, secure, and evaluate system performance  

---

![Travel journal Architecture](/images/2-Proposal/proposal.jpg)


