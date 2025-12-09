---
title: "SQS Pipeline"
date: 2025-12-08
weight: 3
chapter: false
pre: "<b>5.4.3. </b>"
---

## Mục đích

Xử lý ảnh bất đồng bộ để:
- Giảm tải cho hệ thống khi nhiều người upload ảnh cùng lúc
- Đảm bảo không mất message khi Lambda gặp lỗi
- Tách biệt các bước xử lý (moderation → detect labels)

## Kiến trúc SQS Pipeline

```
S3 Upload Event
    ↓
SQS: ModerationQueue
    ↓
Lambda: Content Moderation
    ↓ (if approved)
SQS: DetectLabelsQueue
    ↓
Lambda: Detect Labels
    ↓
DynamoDB + Gallery
```

## Demo: SQS Queues Overview

Hệ thống sử dụng nhiều SQS queues để xử lý ảnh theo pipeline:

![SQS Queues Overview](/images/5-Workshop/5.4-image-processing/SQS_QUESUES_OVERVIEW.png)

## Lợi ích của SQS

### 1. Xử lý bất đồng bộ
- Lambda không cần chờ xử lý xong mới trả về
- Người dùng upload ảnh nhanh chóng
- Xử lý diễn ra ở background

### 2. Retry tự động
- Nếu Lambda lỗi, message quay lại queue
- Tự động retry với backoff
- Dead Letter Queue (DLQ) cho message lỗi nhiều lần

### 3. Scaling tự động
- SQS tự động scale theo số lượng message
- Lambda được trigger song song khi có nhiều message
- Không lo quá tải hệ thống

### 4. Tách biệt các bước
- Content Moderation và Detect Labels độc lập
- Dễ debug và monitor từng bước
- Có thể thêm bước xử lý mới dễ dàng

## Cấu hình Queue

### ModerationQueue
- Visibility Timeout: 300s (5 phút)
- Message Retention: 4 ngày
- DLQ: ModerationDLQ (sau 3 lần retry)

### DetectLabelsQueue
- Visibility Timeout: 180s (3 phút)
- Message Retention: 4 ngày
- DLQ: DetectLabelsDLQ (sau 3 lần retry)

## Monitoring

Theo dõi các metrics quan trọng:
- **ApproximateNumberOfMessagesVisible** - Số message đang chờ
- **ApproximateAgeOfOldestMessage** - Message cũ nhất
- **NumberOfMessagesReceived** - Tổng message nhận được
- **NumberOfMessagesSent** - Tổng message gửi đi

## Kết luận

SQS Pipeline giúp hệ thống:
- Xử lý ổn định và đáng tin cậy
- Scale tự động theo nhu cầu
- Dễ bảo trì và mở rộng
