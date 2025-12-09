---
title: "CloudFront, S3 Upload & Location Service"
date: 2025-12-09
weight: 6
chapter: true
pre: "<b>5.6. </b>"
alwaysopen: false
---

## CloudFront (Static + Dynamic), S3 Upload Web, AWS Location Service

Phần này bao gồm triển khai **Amazon CloudFront** cho phân phối nội dung, **Amazon S3** cho upload ảnh sử dụng pre-signed URLs, và **AWS Location Service** cho geocoding và bản đồ trong Travel Guide Application.

### Tổng quan

Travel Guide application sử dụng ba dịch vụ AWS chính để mang lại trải nghiệm nhanh, an toàn và giàu tính năng:

**Amazon CloudFront:**
- CDN cho nội dung web tĩnh (React build)
- CDN cho ảnh (articles, thumbnails, avatars, covers)
- HTTPS redirect và compression
- Origin Access Identity (OAI) cho truy cập S3 an toàn

**Amazon S3 Upload Web:**
- Pre-signed URLs cho upload trực tiếp an toàn
- Không cần AWS credentials trên client
- Quyền upload giới hạn thời gian (15 phút)
- Cấu hình CORS cho browser uploads

**AWS Location Service:**
- Place Index cho geocoding/reverse geocoding
- Nguồn dữ liệu Esri
- DynamoDB cache để tối ưu chi phí
- Nominatim fallback để tăng độ tin cậy

### Kiến trúc

Hệ thống hoạt động theo ba luồng chính:

#### Luồng A — Web tĩnh
1. Người dùng truy cập ứng dụng React web
2. CloudFront đóng vai trò CDN
3. CloudFront lấy nội dung từ S3 StaticSiteBucket (private) thông qua OAI
4. Người dùng luôn được redirect HTTPS và nhận file index.html mặc định

#### Luồng B — Upload ảnh & hiển thị ảnh
1. Frontend gọi API `/upload-url` để lấy pre-signed URL
2. Frontend upload ảnh trực tiếp lên S3 ArticleImagesBucket
3. Khi hiển thị bài viết/ảnh, frontend dùng domain CloudFront để tải ảnh nhanh hơn
4. CloudFront map các path ảnh về origin ảnh trong S3

#### Luồng C — Map/Geocoding
1. Khi tạo/cập nhật bài viết có tọa độ, backend dùng AWS Location Service Place Index
2. **Reverse geocoding**: lat/lng → tên địa điểm
3. **Forward geocoding**: text → lat/lng (phục vụ search)
4. DynamoDB cache giảm chi phí Location Service
5. Nominatim fallback để tăng độ bền

### Nội dung

- [S3 Pre-signed URL Upload](5.6.1-s3-presigned-upload/)
- [Cấu hình CloudFront CDN](5.6.2-cloudfront-cdn/)
- [Tích hợp AWS Location Service](5.6.3-location-service/)

