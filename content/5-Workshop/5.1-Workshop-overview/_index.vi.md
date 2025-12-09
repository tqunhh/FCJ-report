---
title: " Giới thiệu Workshop"
weight: 1
---


## **Giới thiệu**

Workshop này trình bày tổng quan về **Travel Journal Web Application** được xây dựng trong dự án.  
Mục tiêu chính là giúp người học nắm được quy trình thiết kế và triển khai một hệ thống web hiện đại dựa trên **kiến trúc serverless của AWS**, kết hợp các dịch vụ như **AWS Rekognition**, **AWS Location Service**, **DynamoDB**, và **Amazon S3** để tạo ra một sản phẩm thực tế, khả năng mở rộng cao và tối ưu chi phí.

Nền tảng cho phép người dùng ghi lại hành trình du lịch bằng cách đăng tải hình ảnh, ghi chú địa điểm, tạo bài viết và chia sẻ trải nghiệm cá nhân.  
Khi ảnh được tải lên, **Amazon Rekognition** sẽ phân tích nội dung và tự động gắn nhãn.  
Metadata của bài viết được lưu vào **DynamoDB**, hình ảnh được lưu trữ trong **S3**, và toàn bộ nội dung tĩnh được phân phối qua **CloudFront** để đảm bảo tốc độ truy cập nhanh.

---


## **Tính năng nổi bật – Hiển thị hành trình**

Điểm nhấn của hệ thống là khả năng hiển thị hành trình di chuyển dựa trên vị trí của từng bài viết bằng **AWS Location Service**, mang lại góc nhìn trực quan và sinh động cho trải nghiệm người dùng.

---

## **Kiến trúc Backend Serverless**

Backend sử dụng hoàn toàn kiến trúc serverless:

- **AWS Lambda** – xử lý logic nghiệp vụ  
- **API Gateway** – cung cấp REST API  
- **DynamoDB** – lưu trữ dữ liệu bài viết và người dùng  
- **Pipeline S3 + SQS + Lambda** – xử lý ảnh tải lên  
- **Rekognition** – gắn nhãn tự động  
- **CloudFront** – phân phối nội dung hiệu năng cao  

Kiến trúc này giảm thiểu việc vận hành, tự động mở rộng theo tải và tiết kiệm chi phí.

---


## **Xây dựng Frontend**

Frontend sử dụng React và được triển khai lên:
  
- **AWS CloudFront**

giúp tối ưu hiệu năng và đơn giản hóa triển khai.

---

## **Mục tiêu Workshop**

Sau workshop, người học sẽ hiểu rõ:

- Cách kết hợp nhiều dịch vụ AWS để xây dựng hệ thống web hoàn chỉnh  
- Cách hoạt động của kiến trúc serverless  
- Cơ chế xử lý dữ liệu ảnh bằng AI Rekognition  
- Quy trình triển khai, bảo mật và đánh giá hiệu năng hệ thống  

---
![Travel journal Architecture](/images/2-Proposal/proposal.jpg)
