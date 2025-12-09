---
title: "IAM Roles & Policies"
date: 2025-12-08
weight: 2
chapter: false
pre: "<b>5.5.2. </b>"
---

## IAM - Identity & Access Management

### Overview

IAM is the service for managing access permissions (identity & access). In the Travel Guide workshop, IAM is used to:

- Create users/groups for developers (standard credentials, MFA)
- Create **roles** for Lambda, ECS, EC2 with necessary permissions (least privilege)
- Create **policies** allowing Lambda to access S3/DynamoDB/CloudWatch/Cognito Admin
- Configure trust policies (who can assume roles)
- Grant API Gateway permission to **invoke** Lambda (resource-based permission)
- (Optional) Create roles/permissions for CI/CD

### What is IAM used for in our project?

In the Travel Guide architecture:
- Lambda reads/writes DynamoDB
- Lambda uploads images to S3
- Lambda calls Rekognition to analyze images
- Lambda writes logs to CloudWatch

---

## Preparation Steps

### AWS Account Requirements

**Prerequisites:**
- 1 AWS account
- Permissions to:
  - Create IAM Roles
  - Create Lambda functions
  - Create DynamoDB/S3 resources

![IAM Setup](/images/5-Workshop/5.5-Policy/SetUp_IAM_1.png)

---

## Role for API Gateway

Typically, API Gateway doesn't need a separate IAM role to authorize User Pool. However, for API Gateway to invoke Lambda from integration, you need to add permission to Lambda (resource-based) — usually done via CLI or Console auto-create.

---

## Role for Lambda Functions

**Purpose:** Lambda needs to write logs, access S3/DynamoDB, call Cognito Admin API (if backend manages users).

### Creating Lambda Execution Role

**Console Steps:**
1. IAM → Roles → Create role
2. AWS service → Lambda → Next
3. Attach managed policies:
   - `AWSLambdaBasicExecutionRole` (CloudWatch logs)
   - `AmazonDynamoDBFullAccess` (replace with restricted table-specific policy)
   - `AmazonS3ReadOnlyAccess` or custom S3 policy

![Create Lambda Role](/images/5-Workshop/5.5-Policy/SetUp_IAM_2.png)

![Attach Policies](/images/5-Workshop/5.5-Policy/SetUp_IAM3.png)

---

## DynamoDB Permissions

**Policy for Article CRUD operations:**

```json
{
  "Effect": "Allow",
  "Action": [
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:UpdateItem",
    "dynamodb:DeleteItem",
    "dynamodb:Query"
  ],
  "Resource": "arn:aws:dynamodb:*:*:table/TravelGuide-*"
}
```

**Why these permissions?**
- Lambda creates/edits/deletes articles
- Only access specific tables needed
- Follows least privilege principle

---

## S3 Permissions (Images)

**Policy for image upload/download:**

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::travel-guide-*/\*"
}
```

**Use cases:**
- Upload user-submitted images
- Retrieve images for display
- Process images for thumbnails

---

## Rekognition Permissions

**Policy for image analysis:**

```json
{
  "Effect": "Allow",
  "Action": [
    "rekognition:DetectLabels",
    "rekognition:DetectModerationLabels"
  ],
  "Resource": "*"
}
```

**Use cases:**
- Detect labels in travel photos
- Content moderation for inappropriate images
- Auto-tagging images

---

## CloudWatch Logs Permissions

**Policy for logging:**

```json
{
  "Effect": "Allow",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Resource": "arn:aws:logs:*:*:*"
}
```

**Why needed?**
- Debug Lambda functions
- Monitor application behavior
- Track errors and performance

---

## Complete Lambda Execution Role

**Combined policy example:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:DeleteItem",
        "dynamodb:Query",
        "dynamodb:Scan"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/TravelGuide-*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::travel-guide-*/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "rekognition:DetectLabels",
        "rekognition:DetectModerationLabels"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Trust Policy

**Who can assume this role?**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This allows Lambda service to assume the role and use its permissions.

---

## Best Practices

### 1. Least Privilege Principle
- Only grant permissions actually needed
- Avoid using `*` in Resource ARNs when possible
- Use specific actions instead of `*`

### 2. Separate Roles by Function
- Different Lambda functions = different roles
- Article service role ≠ Media service role
- Easier to audit and manage

### 3. Use Managed Policies When Appropriate
- `AWSLambdaBasicExecutionRole` for logging
- Custom policies for business logic
- Combine managed + custom policies

### 4. Regular Audits
- Review unused permissions
- Check for overly permissive policies
- Use IAM Access Analyzer

### 5. Tag Resources
```json
{
  "Tags": [
    {
      "Key": "Service",
      "Value": "ArticleService"
    },
    {
      "Key": "Environment",
      "Value": "staging"
    }
  ]
}
```

---

## Resource-Based Permissions

### API Gateway Invoke Lambda

**Lambda resource policy:**

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "apigateway.amazonaws.com"
  },
  "Action": "lambda:InvokeFunction",
  "Resource": "arn:aws:lambda:*:*:function:CreateArticle",
  "Condition": {
    "ArnLike": {
      "AWS:SourceArn": "arn:aws:execute-api:*:*:*/*/POST/articles"
    }
  }
}
```

This allows API Gateway to invoke the Lambda function.

---

## Key Takeaways

1. **IAM Roles** define what AWS services can do
2. **Policies** specify exact permissions
3. **Trust Policies** define who can assume roles
4. **Least Privilege** is critical for security
5. **Separate roles** for different services
6. **Regular audits** prevent permission creep
7. **Resource-based policies** for cross-service access

---

## Common Pitfalls

❌ **Using Administrator Access**
```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

✅ **Use Specific Permissions**
```json
{
  "Effect": "Allow",
  "Action": ["dynamodb:GetItem", "dynamodb:PutItem"],
  "Resource": "arn:aws:dynamodb:*:*:table/Articles"
}
```

❌ **Sharing Roles Across Services**
- One role for all Lambda functions

✅ **Separate Roles**
- ArticleServiceRole
- MediaServiceRole
- AIServiceRole
