---
title: "IAM Roles & Policies"
date: 2025-12-08
weight: 2
chapter: false
pre: "<b>5.5.2. </b>"
---

## IAM - Identity & Access Management

### Tổng quan

IAM là dịch vụ quản lý quyền truy cập (identity & access). Trong workshop Travel Guide, IAM dùng để:

- Tạo người dùng/nhóm cho developers (credentials chuẩn, MFA)
- Tạo **roles** cho Lambda, ECS, EC2 với quyền cần thiết (least privilege)
- Tạo **policies** cho phép Lambda truy cập S3/DynamoDB/CloudWatch/Cognito Admin
- Cấu hình trust policies (ai được assume roles)
- Cấp quyền để API Gateway **invoke** Lambda (resource-based permission)
- (Tùy chọn) Tạo roles/permissions cho CI/CD

### IAM dùng để làm gì trong dự án của chúng ta?

Trong kiến trúc Travel Guide:
- Lambda đọc/ghi DynamoDB
- Lambda upload ảnh lên S3
- Lambda gọi Rekognition phân tích ảnh
- Lambda ghi log CloudWatch

---

## Các bước Chuẩn bị

### Yêu cầu AWS Account

**Điều kiện tiên quyết:**
- 1 AWS account
- Quyền để:
  - Tạo IAM Roles
  - Tạo Lambda functions
  - Tạo DynamoDB/S3 resources

![Thiết lập IAM](/images/5-Workshop/5.5-Policy/SetUp_IAM_1.png)

---

## Role cho API Gateway

Thông thường, API Gateway không cần một IAM role riêng để authorize User Pool. Tuy nhiên, để API Gateway invoke Lambda từ integration, bạn cần thêm permission vào Lambda (resource-based) — thường làm bằng CLI hoặc Console auto-create.

---

## Role cho Lambda Functions

**Mục đích:** Lambda cần ghi log, truy cập S3/DynamoDB, gọi Cognito Admin API (nếu backend quản lý users).

### Tạo Lambda Execution Role

**Các bước trên Console:**
1. IAM → Roles → Create role
2. AWS service → Lambda → Next
3. Attach managed policies:
   - `AWSLambdaBasicExecutionRole` (CloudWatch logs)
   - `AmazonDynamoDBFullAccess` (thay bằng policy hạn chế table cụ thể)
   - `AmazonS3ReadOnlyAccess` hoặc custom S3 policy

![Tạo Lambda Role](/images/5-Workshop/5.5-Policy/SetUp_IAM_2.png)

![Gắn Policies](/images/5-Workshop/5.5-Policy/SetUp_IAM3.png)

---

## Quyền DynamoDB

**Policy cho thao tác CRUD bài viết:**

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

**Tại sao cần những quyền này?**
- Lambda tạo/sửa/xóa bài viết
- Chỉ truy cập đúng tables cần dùng
- Tuân theo nguyên tắc least privilege

---

## Quyền S3 (Ảnh)

**Policy cho upload/download ảnh:**

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:PutObject",
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::travel-guide-*/*"
}
```

**Trường hợp sử dụng:**
- Upload ảnh do người dùng gửi
- Lấy ảnh để hiển thị
- Xử lý ảnh cho thumbnails

---

## Quyền Rekognition

**Policy cho phân tích ảnh:**

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

**Trường hợp sử dụng:**
- Phát hiện labels trong ảnh du lịch
- Kiểm duyệt nội dung ảnh không phù hợp
- Tự động gắn tags cho ảnh

---

## Quyền CloudWatch Logs

**Policy cho logging:**

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

**Tại sao cần?**
- Debug Lambda functions
- Monitor hành vi ứng dụng
- Theo dõi lỗi và hiệu suất

---

## Lambda Execution Role Hoàn chỉnh

**Ví dụ policy kết hợp:**

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

**Ai có thể assume role này?**

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

Điều này cho phép Lambda service assume role và sử dụng permissions của nó.

---

## Best Practices

### 1. Nguyên tắc Least Privilege
- Chỉ cấp quyền thực sự cần thiết
- Tránh dùng `*` trong Resource ARNs khi có thể
- Dùng actions cụ thể thay vì `*`

### 2. Tách Roles theo Chức năng
- Lambda functions khác nhau = roles khác nhau
- Article service role ≠ Media service role
- Dễ audit và quản lý hơn

### 3. Dùng Managed Policies khi Phù hợp
- `AWSLambdaBasicExecutionRole` cho logging
- Custom policies cho business logic
- Kết hợp managed + custom policies

### 4. Audit Thường xuyên
- Review quyền không sử dụng
- Kiểm tra policies quá rộng
- Dùng IAM Access Analyzer

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

Điều này cho phép API Gateway invoke Lambda function.

---

## Điểm chính

1. **IAM Roles** định nghĩa AWS services có thể làm gì
2. **Policies** chỉ định quyền chính xác
3. **Trust Policies** định nghĩa ai có thể assume roles
4. **Least Privilege** quan trọng cho bảo mật
5. **Tách roles** cho các services khác nhau
6. **Audit thường xuyên** ngăn permission creep
7. **Resource-based policies** cho cross-service access

---

## Lỗi Thường gặp

❌ **Dùng Administrator Access**
```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

✅ **Dùng Quyền Cụ thể**
```json
{
  "Effect": "Allow",
  "Action": ["dynamodb:GetItem", "dynamodb:PutItem"],
  "Resource": "arn:aws:dynamodb:*:*:table/Articles"
}
```

❌ **Chia sẻ Roles giữa Services**
- Một role cho tất cả Lambda functions

✅ **Tách Roles**
- ArticleServiceRole
- MediaServiceRole
- AIServiceRole
