---
title: "Backend - Dịch vụ Bài viết"
date: 2025-12-08
weight: 3
chapter: true
pre: "<b>5.3. </b>"
alwaysopen: false
---

#### Tổng quan Kiến trúc Backend

Phần này bao gồm toàn bộ triển khai backend cho Hệ thống Quản lý Bài viết. Backend được xây dựng sử dụng các dịch vụ serverless của AWS bao gồm Lambda, DynamoDB, S3, Cognito và AWS Location Service.

Kiến trúc hỗ trợ:
- Các thao tác CRUD đầy đủ cho bài viết
- Tải lên và quản lý hình ảnh
- Tính năng yêu thích và tương tác của người dùng
- Tính năng gallery và xu hướng
- Dịch vụ dựa trên vị trí với caching
- Quản lý hồ sơ người dùng

#### Nội dung

- [Lambda Service Backend](5.3.1-lambda-service-backend/)
- [Bảng DynamoDB](5.3.2-dynamotable/)