---
title: "Cognito & IAM - Xác thực & Phân quyền"
date: 2025-12-08
weight: 5
chapter: true
pre: "<b>5.5. </b>"
alwaysopen: false
---

## Xác thực An toàn cho Ứng dụng Web/Mobile

Phần này bao gồm triển khai **AWS Cognito** cho xác thực người dùng và **IAM** cho phân quyền trong Travel Guide Application.

### Tổng quan

AWS Cognito cung cấp dịch vụ quản lý người dùng và xác thực cho ứng dụng web/mobile mà không cần tự xây dựng hệ thống đăng nhập từ đầu. Kết hợp với IAM roles và policies, nó tạo ra một hệ thống xác thực và phân quyền an toàn, có khả năng mở rộng.

### Các thành phần chính

**AWS Cognito:**
- User Pool để quản lý người dùng
- Đăng ký / Đăng nhập / Xác thực OTP email
- Quản lý JWT Token (id_token, access_token, refresh_token)
- Tích hợp Frontend (React)
- Tích hợp Backend (API Gateway + Cognito Authorizer)

**IAM (Identity & Access Management):**
- Roles cho Lambda, ECS, EC2
- Policies cho quyền truy cập S3, DynamoDB, CloudWatch
- Trust policies và resource-based permissions
- Nguyên tắc least privilege

### Kiến trúc

Luồng xác thực:
1. Người dùng đăng ký/đăng nhập qua Cognito Hosted UI
2. Cognito phát hành JWT tokens
3. Frontend lưu trữ và quản lý tokens
4. API Gateway xác thực tokens bằng Cognito Authorizer
5. Lambda functions truy cập AWS resources bằng IAM roles

### Nội dung

- [Cấu hình AWS Cognito](5.5.1-cognito-setup/)
- [IAM Roles & Policies](5.5.2-iam-roles-policies/)
