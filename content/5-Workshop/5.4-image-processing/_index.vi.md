---
title: "Xử lý Ảnh - Lambda, Rekognition, SQS, SNS, SES"
date: 2025-12-08
weight: 4
chapter: false
pre: "<b>5.4. </b>"
---

#### Tổng quan

Trong dự án Travel Guide, hệ thống xử lý ảnh tự động bao gồm các thành phần chính:

- **Lambda Content Moderation** - Kiểm duyệt nội dung ảnh
- **Lambda Detect Labels** - Phát hiện và gắn nhãn tự động
- **SQS Queue** - Xử lý bất đồng bộ
- **SNS Topic & SES** - Thông báo qua email

#### Kiến trúc Lambda Functions

![Toàn bộ Lambda Functions](/images/5-Workshop/5.4-image-processing/Full_lamdba_Function.png)

#### Luồng xử lý

1. Người dùng upload ảnh → S3
2. S3 Event → SQS Queue
3. Lambda Content Moderation kiểm tra nội dung
4. Nếu hợp lệ → Lambda Detect Labels gắn nhãn
5. Nếu vi phạm → SNS + SES gửi email cảnh báo

#### Nội dung

- [Lambda Content Moderation](5.4.1-content-moderation/)
- [Lambda Detect Labels](5.4.2-detect-labels/)
- [SQS Pipeline](5.4.3-sqs-pipeline/)
- [SNS & SES Notification](5.4.4-sns-ses-notification/)
