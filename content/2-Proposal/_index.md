---
title: "Proposal"
date: "2025-10-10"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---


# Travel Journal
## A Unified AWS Serverless Solution for Travel Journal


### 1. Executive Summary
Travel Journal Web was created to help people preserve their life journeys — not only the trips they take but also the memories, emotions, and stories behind each photo. The application connects people to their experiences, turning every journey into part of their own “memory map.”

Users can upload photos, add location notes, and the system automatically recognizes the scene type (beach, mountain, city, etc.) using Amazon Rekognition. Travel routes are displayed visually and in real-time on an interactive map powered by Amazon Location Service, delivering a vivid and engaging experience.

The platform leverages the power of AWS Serverless Architecture — including Lambda, API Gateway, S3, DynamoDB, and Cognito — ensuring high performance, strong security, and scalable flexibility at an optimized cost.

### 2. Problem Statement
### What’s the Problem?
Many travel enthusiasts want to document their journeys, yet existing platforms only allow posting scattered images or notes without an intuitive connection between emotions, photos, and actual locations. It becomes difficult to organize memories, view trips on a map, or analyze travel data such as destinations, durations, and experiences. Moreover, popular cloud storage solutions fail to personalize the user experience.

### The Solution
Travel Journal Web is built as a web application leveraging the AWS Serverless architecture to optimize performance and cost efficiency.

The system uses Amazon S3 to store images and static data, distributed globally through Amazon CloudFront. Amazon Cognito manages secure user authentication, while API Gateway and AWS Lambda handle backend logic. Uploaded images in S3 are analyzed by Amazon Rekognition, with results and location data stored in Amazon DynamoDB and visualized using Amazon Location Service.

The entire system is monitored by Amazon CloudWatch, tracking throughput, errors, latency, and database capacity; alerts are sent via SNS. Data and encryption keys are securely managed using AWS KMS and Secrets Manager.

This solution delivers a smart, secure, and cost-effective travel application, allowing users to easily capture, store, and revisit their journeys anytime, anywhere..

### Benefits and Return on Investment
The solution allows users to easily record and share their journeys while establishing a data foundation for potential expansion into a social travel platform. With an estimated cost of only USD 14.55/month, the app can support 100–200 users without physical servers. The expected ROI period is 6–8 months, achieved by saving maintenance and storage costs.

### 3. Solution Architecture
The Travel Journal Web is built entirely on an AWS Serverless architecture, optimizing performance, security, and scalability. The static web interface is hosted on Amazon S3, distributed globally through Amazon CloudFront, and protected by AWS WAF, ACM, and Route 53. User authentication is managed by Amazon Cognito, while Amazon API Gateway and AWS Lambda handle backend business logic. Uploaded images are stored in Amazon S3 and automatically processed through Amazon SQS, AWS Lambda, Amazon Rekognition, and Amazon Location Service. The results are stored in Amazon DynamoDB and redistributed via S3. The system supports retry and DLQ mechanisms, sends notifications through Amazon SNS, provides centralized monitoring with Amazon CloudWatch and AWS X-Ray, and secures data using AWS IAM, KMS, and Secrets Manager.. The architecture is detailed below:

![Travel journal Architecture](/images/travel_journal_architecture.jpg)


### AWS Services Used
- **Amazon Route 53**: Global domain management and routing.
- **AWS Certificate Manager** (ACM): Issues and manages SSL/TLS certificates for secure endpoints
- **Amazon CloudFront**: Low-latency content delivery for static and dynamic assets.
- **AWS WAF**: Protects the application from common web threats.
- **AWS Lambda**: Event-driven serverless compute for backend logic.
- **Amazon API Gateway**: Acts as the interface between the frontend and Lambda backend.
- **Amazon S3**: Stores user data, images, and activity logs
- **Amazon DynamoDB**: NoSQL database for trip records and user data, optimized for fast queries.
- **Amazon Cognito**: User authentication and authorization management.
- **Amazon Rekognition**: Image analysis and labeling.
- **Amazon Location Service**: Geolocation and map visualization.
- **Amazon SNS**: Sends notifications to users and administrators.
- **Amazon SQS (Main Queue)**: Buffers image processing requests from S3 uploads before invoking Lambda.
- **Dead Letter Queue (DLQ)**: Captures failed or unprocessed messages from SQS for later review and troubleshooting.
- **Amazon CloudWatch**: Logs, metrics, and performance monitoring.
- **AWS IAM**: Manages access roles and permissions for AWS services.
- **AWS KMS**: Encrypts data at rest and in transit, enhancing overall security.
- **AWS Secrets Manager**: Securely stores and encrypts confidential credentials.
- **AWS CodeBuild**: Automatically compiles, tests, and packages source code.
- **AWS CodePipeline**: Automates the entire CI/CD process — from commit, build, and test to deployment in AWS environments.

### Component Design
- **User Authentication**: Managed by Amazon Cognito for login, token management, and access control.
- **Application Logic**: AWS Lambda handles travel data submissions, image uploads, and analytics requests from API Gateway.
- **Data Management**: Amazon DynamoDB stores journey details; OpenSearch provides fast indexing and search.
- **Queue Processing**: Amazon SQS (Main Queue) receives image processing requests from S3 before invoking Lambda; the Dead Letter Queue (DLQ) captures failed messages for later handling.
- **Image Analysis**: Amazon Rekognition automatically labels and classifies photo content.
- **Maps & Location Data**: Amazon Location Service tracks and displays GPS data on interactive maps.
- **Content Storage & Delivery**: Amazon S3 stores images, user data, and static assets; content is distributed globally via Amazon CloudFront (protected by AWS WAF, SSL/TLS via ACM, and routed through Route 53).
- **Monitoring & Notifications**: Amazon CloudWatch monitors logs and performance metrics; Amazon SNS sends alerts and notifications to users.
- **Security & Access Management**: AWS IAM manages service permissions, while AWS Secrets Manager and KMS protect sensitive information.
- **Deployment & CI/CD**: AWS CodeBuild and CodePipeline automate the build, test, and deployment processes.

### 4. Technical Implementation
**Implementation Phases**
The project is divided into two main areas — web application development and AWS infrastructure integration — across three stages:
- Requirement Analysis & Architecture Design: Identify suitable AWS services (CloudFront, WAF, Cognito, DynamoDB, Lambda, API Gateway, S3, etc.) and design the overall architecture (Month 1).
- Cost Estimation & System Simulation: Calculate costs using AWS Pricing Calculator, test image and location processing workflows, and optimize system resources (Month 2).
- Development, Testing & Deployment: Build the web application, integrate AWS SDKs, test WAF protection, and monitor operations using CloudWatch (Month 3).

**Technical Requirements**
- Frontend: Built with React.js, deployed as static files on Amazon S3, and globally distributed via CloudFront for fast load times and reduced backend load.
- Security & Access: AWS WAF protects against common web attacks; AWS Certificate Manager (ACM) provides SSL/TLS certificates; Amazon Cognito manages user authentication (email, OAuth2).
- Backend: API Gateway routes user requests to AWS Lambda, which executes business logic such as trip logging, image uploads, and data queries.
- Queue Processing: Amazon SQS (Main Queue) buffers image processing requests triggered from S3, while the Dead Letter Queue (DLQ) stores failed messages for later handling.
- Database & Search: Amazon DynamoDB stores user journeys and notes
- Image Analysis: Amazon Rekognition detects scenes, faces, and automatically suggests image labels.
- Geolocation & Maps: Amazon Location Service visualizes GPS coordinates on an interactive map.
- Monitoring & Security: Amazon CloudWatch collects logs; AWS Secrets Manager and KMS secure sensitive information
- Notifications: Amazon SNS sends alerts for system errors or new user events.
- Deployment & CI/CD: AWS CodeBuild and AWS CodePipeline automate the build, test, and deployment workflow.

### 5. Timeline & Milestones
**Project Timeline**
- Internship (Months 1-3): 3 months.
    - Month 1: Learn AWS fundamentals.
    - Month 2: Design architecture, estimate costs, and refine infrastructure.
    - Month 3: Implement, test, and deploy the application.
- Post-Launch:Continue system research and improvements over the following year.
### 6. Budget Estimation
You can find the budget estimation on the [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=e732e6253c20390cbb8d794f05b4a951dec9d574) 

### Infrastructure Costs
- AWS Services:
    - Amazon Route 53: 0,50 USD/month
    - AWS Certificate Manager: 0,0 USD/month
    - Amazon CloudFront: 0,61 USD/month
    - AWS WAF: 0,6 USD/month
    - AWS Lambda: 0,01 USD/month
    - Amazon API Gateway: 0,45 USD/month
    - Amazon S3: 1,47 USD/month
    - Amazon DynamoDB: 16,35 USD/month
    - Amazon Cognito: 5,00 USD/month
    - Amazon Rekognition: 10,08 USD/month
    - Amazon Location Service: 4,35 USD/month
    - Amazon SNS: 2,58 USD/month
    - Amazon SQS (Main Queue): 3,1 USD/month
    - Amazon CloudWatch: 6,87 USD/month
    - AWS IAM: 0,00 USD/month
    - AWS Secrets Manager: 0,4 USD/month
    - AWS KMS: 2,3 USD/month
    - AWS CodeBuild: 0,8 USD/month
    - AWS CodePipeline: 0,00 USD/month
  
*Total*: 61.78 USD/month, or $946,56 USD/year

### 7. Risk Assessment
#### Risk Matrix
- Security breach or user access loss: High impact, very low probability.
- Cost increase due to Rekognition usage: High impact, medium probability.
- GPS data inconsistency: High impact, low probability

#### Mitigation Strategies
- Network: Use CloudFront for global caching and stable performance across regions.
- Infrastructure: Leverage Lambda’s automatic retry mechanism and browser caching to minimize downtime.
- Security: Apply multi-layer authentication via Cognito and strict IAM/S3 permissions.
- Cost: Set budget alerts in AWS Budgets, and optimize Lambda/S3 usage based on access patterns.

#### Contingency Plans
- Integrate SNS and CloudWatch Alerts to notify admins immediately of failures (e.g., Lambda errors, API overloads, budget overruns).
- Use CloudFormation for rapid infrastructure recovery
- Enable S3 Versioning to preserve image and log backups.

### 8. Expected Outcomes
#### Technical Improvements: 
The app automates storage, analysis, and visualization of travel data on a map, eliminating manual note-taking.
#### Long-term Value
Builds a rich dataset of travel routes, photos, and emotions — forming a foundation for future applications like personalized travel recommendations and experience analytics.