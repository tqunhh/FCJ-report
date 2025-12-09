---
title: "Cognito & IAM - Authentication & Authorization"
date: 2025-12-08
weight: 5
chapter: true
pre: "<b>5.5. </b>"
alwaysopen: false
---

## Secure Authentication for Web/Mobile Applications

This section covers the implementation of **AWS Cognito** for user authentication and **IAM** for authorization in the Travel Guide Application.

### Overview

AWS Cognito provides user management and authentication services for web/mobile applications without building a login system from scratch. Combined with IAM roles and policies, it creates a secure, scalable authentication and authorization system.

### Key Components

**AWS Cognito:**
- User Pool for user management
- Sign-up / Sign-in / OTP email verification
- JWT Token management (id_token, access_token, refresh_token)
- Frontend integration (React)
- Backend integration (API Gateway + Cognito Authorizer)

**IAM (Identity & Access Management):**
- Roles for Lambda, ECS, EC2
- Policies for S3, DynamoDB, CloudWatch access
- Trust policies and resource-based permissions
- Least privilege principle

### Architecture

The authentication flow:
1. User signs up/signs in via Cognito Hosted UI
2. Cognito issues JWT tokens
3. Frontend stores and manages tokens
4. API Gateway validates tokens using Cognito Authorizer
5. Lambda functions access AWS resources using IAM roles

### Content

- [AWS Cognito Setup](5.5.1-cognito-setup/)
- [IAM Roles & Policies](5.5.2-iam-roles-policies/)
