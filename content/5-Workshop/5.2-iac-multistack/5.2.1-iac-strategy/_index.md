---
title: "IaC Strategy & Tool Selection"
date: 2025-12-08
weight: 1
chapter: false
pre: "<b>5.2.1. </b>"
---

## Infrastructure as Code Strategy

### Why Infrastructure as Code?

Infrastructure as Code (IaC) allows us to:
- **Version Control**: Track infrastructure changes in Git
- **Reproducibility**: Deploy identical environments consistently
- **Automation**: Reduce manual errors and deployment time
- **Documentation**: Code serves as living documentation
- **Collaboration**: Team members can review and contribute

### Why CloudFormation/SAM?

For the Travel Guide Application, we chose **AWS CloudFormation** with **SAM (Serverless Application Model)** for the following reasons:

#### Advantages

**1. Native AWS Integration**
- First-class support for all AWS services
- No additional state management required
- Automatic rollback on failures
- Built-in drift detection

**2. SAM Simplification**
- Simplified syntax for Lambda, API Gateway, DynamoDB
- Local testing capabilities (`sam local`)
- Built-in best practices for serverless
- Automatic IAM role generation

**3. No Additional Cost**
- CloudFormation is free (pay only for resources)
- No need for external state storage
- No licensing fees

**4. AWS Ecosystem**
- Integrates with CodePipeline, CodeBuild
- CloudWatch integration for monitoring
- AWS Console visualization

#### Trade-offs vs Alternatives

| Feature | CloudFormation/SAM | Terraform | AWS CDK | Pulumi |
|---------|-------------------|-----------|---------|--------|
| **Learning Curve** | Medium | Medium | High | High |
| **Syntax** | YAML/JSON | HCL | TypeScript/Python | TypeScript/Python/Go |
| **State Management** | AWS-managed | Manual/Cloud | AWS-managed | Cloud |
| **Multi-Cloud** | ❌ No | ✅ Yes | ❌ No | ✅ Yes |
| **Cost** | Free | Free (Cloud paid) | Free | Free (Cloud paid) |
| **AWS Integration** | Native | Good | Native | Good |
| **Serverless Support** | Excellent (SAM) | Good | Excellent | Good |

### Template Organization Strategy

Our templates are organized as follows:

```
infrastructure/
├── core/
│   └── template.yaml          # Core stack (DynamoDB, S3, Cognito)
├── services/
│   ├── auth/
│   │   └── template.yaml      # Auth service Lambda functions
│   ├── articles/
│   │   └── template.yaml      # Article service Lambda functions
│   ├── media/
│   │   └── template.yaml      # Media processing Lambda functions
│   └── ...
├── parameters/
│   ├── staging.json           # Staging environment parameters
│   └── prod.json              # Production environment parameters
└── scripts/
    ├── deploy.sh              # Main deployment orchestration
    ├── deploy-core.sh         # Core stack deployment
    └── deploy-service.sh      # Service stack deployment
```

### Basic CloudFormation Template Structure

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Travel Guide - Core Infrastructure

Parameters:
  Environment:
    Type: String
    Default: staging
    AllowedValues: [staging, prod]

Resources:
  # DynamoDB Tables
  ArticlesTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub '${AWS::StackName}-articles'
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: articleId
          AttributeType: S
      KeySchema:
        - AttributeName: articleId
          KeyType: HASH

Outputs:
  ArticlesTableName:
    Description: Articles DynamoDB Table Name
    Value: !Ref ArticlesTable
    Export:
      Name: !Sub '${AWS::StackName}-ArticlesTableName'
```

### SAM Transform Example

SAM simplifies Lambda and API Gateway definitions:

```yaml
# Without SAM (CloudFormation only)
CreateArticleFunction:
  Type: AWS::Lambda::Function
  Properties:
    FunctionName: create-article
    Runtime: python3.11
    Handler: index.handler
    Code:
      S3Bucket: my-bucket
      S3Key: function.zip
    Role: !GetAtt LambdaRole.Arn

# With SAM
CreateArticleFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: ./src
    Handler: create_article.handler
    Runtime: python3.11
    Events:
      CreateArticle:
        Type: Api
        Properties:
          Path: /articles
          Method: post
```

### Key Takeaways

1. **CloudFormation/SAM** is ideal for AWS-only deployments
2. **SAM** significantly reduces boilerplate for serverless applications
3. **Native integration** eliminates state management complexity
4. **Template organization** is crucial for maintainability
5. **Trade-offs** exist - multi-cloud requires different tools

### When to Consider Alternatives

- **Terraform**: If you need multi-cloud support or team is already familiar
- **AWS CDK**: If you prefer programming languages over YAML
- **Pulumi**: If you want full programming language power with multi-cloud
