---
title: "Lambda Content Moderation"
date: 2025-12-08
weight: 1
chapter: false
pre: "<b>5.4.1. </b>"
---

## Mục đích

Kiểm duyệt nội dung ảnh tự động để phát hiện vi phạm như:
- Nội dung nhạy cảm/gợi dục
- Bạo lực
- Ma túy/rượu bia
- Biểu tượng xúc phạm

## Giải thích code chính

### a. Nhận message từ SQS

Trích xuất thông tin bucket và key từ S3 event:

```python
for sqs_record in event.get('Records', []):
    try:
        # Parse S3 event from SQS body
        s3_event = json.loads(sqs_record['body'])
        
        # Process each S3 record
        for s3_record in s3_event.get('Records', []):
            try:
                bucket = s3_record['s3']['bucket']['name']
                key = s3_record['s3']['object']['key']
```

### b. Gọi Rekognition detect_moderation_labels

AWS Rekognition phân tích ảnh và trả về danh sách nội dung vi phạm:

```python
moderation_result = moderate_image(bucket, key)

if 'error' in moderation_result:
    print(f"Moderation failed: {moderation_result['error']}")
    results['errors'] += 1
    continue

results['processed'] += 1

if moderation_result['passed']:
    results['approved'] += 1
    mark_article_as_approved(article_id)
    
    # Only forward to next queue if approved
    final_status = {
        'moderationStatus': 'approved',
        'processed': True
    }
    forward_to_next_queue(bucket, key, article_id, final_status)
else:
    results['rejected'] += 1
    
    action_result, owner_id = handle_moderation_failure(
        bucket, key, article_id, moderation_result
    )
    
    if action_result in results['actions']:
        results['actions'][action_result] += 1
```

### c. Xử lý vi phạm - Publish SNS để kích hoạt SES gửi email

Nếu nội dung vi phạm, gửi thông báo cho admin:

```python
# Always send admin notification for deleted/quarantined content
if action_result in ['deleted', 'quarantined']:
    print(f"📧 Sending admin notification for {action_result} content")
    send_admin_notification(article_id, key, moderation_result, owner_id)
elif moderation_result['maxSeverity'] in ['critical', 'high']:
    print(f"📧 Sending admin notification for {moderation_result['maxSeverity']} severity")
    send_admin_notification(article_id, key, moderation_result, owner_id)
    
# DO NOT forward to next queue if rejected/deleted
# Pipeline stops here, user already received deletion email
print(f"⚠️ Pipeline stopped for rejected image: {key}")
print(f"   User notification already sent via send_user_deletion_email()")
```

## Demo: Email thông báo nội dung bạo lực

Khi phát hiện nội dung bạo lực, hệ thống tự động gửi email thông báo:

![Email thông báo - Nội dung bạo lực](/images/5-Workshop/5.4-image-processing/Mail_Violent.png)

## Luồng xử lý

1. Ảnh được upload lên S3
2. S3 event → SQS Queue
3. Lambda đọc từ SQS
4. Rekognition phân tích nội dung
5. Nếu hợp lệ → Chuyển sang queue Detect Labels
6. Nếu vi phạm → Gửi thông báo + Dừng pipeline

## Kết quả

- Ảnh hợp lệ tiếp tục sang bước gắn nhãn
- Ảnh vi phạm bị cách ly/xóa
- Admin và người dùng nhận email thông báo
