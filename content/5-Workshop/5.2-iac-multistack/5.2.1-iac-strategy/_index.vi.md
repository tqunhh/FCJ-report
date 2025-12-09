---
title: "Chiến lược IaC & Lựa chọn công cụ"
date: 2025-12-08
weight: 1
chapter: false
pre: "<b>5.2.1. </b>"
---

## Chiến lược Infrastructure as Code

### Tại sao Infrastructure as Code?

Infrastructure as Code (IaC) cho phép chúng ta:
- **Version Control**: Theo dõi thay đổi hạ tầng trong Git
- **Reproducibility**: Triển khai môi trường giống hệt nhau một cách nhất quán
- **Automation**: Giảm lỗi thủ công và thời gian triển khai
- **Documentation**: Code là tài liệu sống
- **Collaboration**: Thành viên team có thể review và đóng góp

### Tại sao chọn CloudFormation/SAM?

Đối với Travel Guide Application, chúng tôi chọn **AWS CloudFormation** với **SAM (Serverless Application Model)** vì các lý do sau:

#### Ưu điểm

**1. Tích hợp Native với AWS**
- Hỗ trợ đầy đủ cho tất cả AWS services
- Không cần quản lý state bổ sung
- Tự động rollback khi lỗi
- Drift detection tích hợp sẵn

**2. SAM đơn giản hóa**
- Cú pháp đơn giản cho Lambda, API Gateway, DynamoDB
- Khả năng test local (`sam local`)
- Best practices tích hợp sẵn cho serverless
- Tự động tạo IAM roles

**3. Không tốn chi phí bổ sung**
- CloudFormation miễn phí (chỉ trả cho resources)
- Không cần lưu trữ state bên ngoài
- Không có phí license

**4. AWS Ecosystem**
- Tích hợp với CodePipeline, CodeBuild
- CloudWatch integration để monitoring
- Visualization trên AWS Console

#### So sánh với các công cụ khác

| Tính năng | CloudFormation/SAM | Terraform | AWS CDK | Pulumi |
|-----------|-------------------|-----------|---------|--------|
| **Độ khó học** | Trung bình | Trung bình | Cao | Cao |
| **Cú pháp** | YAML/JSON | HCL | TypeScript/Python | TypeScript/Python/Go |
| **Quản lý State** | AWS-managed | Manual/Cloud | AWS-managed | Cloud |
| **Multi-Cloud** | ❌ Không | ✅ Có | ❌ Không | ✅ Có |
| **Chi phí** | Miễn phí | Miễn phí (Cloud trả phí) | Miễn phí | Miễn phí (Cloud trả phí) |
| **Tích hợp AWS** | Native | Tốt | Native | Tốt |
| **Hỗ trợ Serverless** | Xuất sắc (SAM) | Tốt | Xuất sắc | Tốt |

### Chiến lược tổ chức Templates

Templates được tổ chức như sau:

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

### Cấu trúc cơ bản CloudFormation Template

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

### Ví dụ SAM Transform

SAM đơn giản hóa định nghĩa Lambda và API Gateway:

```yaml
# Không dùng SAM (chỉ CloudFormation)
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

# Với SAM
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

### Điểm chính

1. **CloudFormation/SAM** lý tưởng cho triển khai chỉ trên AWS
2. **SAM** giảm đáng kể boilerplate cho serverless applications
3. **Tích hợp native** loại bỏ độ phức tạp quản lý state
4. **Tổ chức template** rất quan trọng cho khả năng bảo trì
5. **Trade-offs** tồn tại - multi-cloud cần công cụ khác

### Khi nào nên xem xét các công cụ khác

- **Terraform**: Nếu cần multi-cloud hoặc team đã quen
- **AWS CDK**: Nếu thích ngôn ngữ lập trình hơn YAML
- **Pulumi**: Nếu muốn sức mạnh ngôn ngữ lập trình đầy đủ với multi-cloud
