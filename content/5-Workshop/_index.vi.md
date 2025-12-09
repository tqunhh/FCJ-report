---
title: "Workshop"
date: 2025-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


# Xây dựng Ứng dụng Travel Guide với AWS Serverless

#### Tổng quan

Workshop này hướng dẫn bạn xây dựng một **Ứng dụng Travel Guide** hoàn chỉnh sử dụng các dịch vụ AWS Serverless. Bạn sẽ học cách thiết kế, triển khai và deploy một ứng dụng production-ready với các mẫu kiến trúc cloud hiện đại.

Ứng dụng cho phép người dùng chia sẻ trải nghiệm du lịch, tải lên ảnh, và khám phá các điểm đến thông qua hệ thống gallery được hỗ trợ bởi AI.

**Công nghệ chính:**
- **Backend:** AWS Lambda, API Gateway, DynamoDB
- **Xử lý AI:** Amazon Rekognition, SQS, SNS/SES
- **Xác thực:** Amazon Cognito, IAM
- **Lưu trữ & CDN:** S3, CloudFront
- **Hạ tầng:** AWS SAM, CloudFormation

#### Nội dung

1. [Tổng quan Workshop](5.1-workshop-overview/)
2. [Infrastructure as Code & Kiến trúc Multi-Stack](5.2-iac-multistack/)
3. [Backend API & Dịch vụ Articles](5.3-backend-articles/)
4. [Pipeline Xử lý Ảnh AI](5.4-image-processing/)
5. [Xác thực với Cognito & IAM](5.5-auth-cognito-iam/)
6. [CloudFront CDN & Dịch vụ Location](5.6-cloudfront-s3-location/)
7. [Triển khai Bảo mật](5.7-security/)