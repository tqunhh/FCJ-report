---
title: "SNS & SES Notification"
date: 2025-12-08
weight: 4
chapter: false
pre: "<b>5.4.4. </b>"
---

## Mục đích

Gửi thông báo email tự động khi:
- Ảnh vi phạm nội dung được phát hiện
- Admin cần được thông báo về nội dung vi phạm
- Người dùng cần biết ảnh của họ bị xóa/cách ly

## Kiến trúc SNS & SES

```
Lambda Content Moderation
    ↓ (if violation detected)
SNS Topic: ImageModerationAlerts
    ↓
    ├─→ SES: Email to Admin
    └─→ SES: Email to User
```

## Demo: Cấu hình SNS Topic

SNS Topic được cấu hình để phân phối thông báo đến nhiều subscribers:

![Cấu hình SNS Topic](/images/5-Workshop/5.4-image-processing/SNS.png)

## Demo: SES Verified Emails

Trước khi gửi email, địa chỉ phải được xác thực trong SES:

![SES Verified Emails](/images/5-Workshop/5.4-image-processing/SES_Verified_email.png)

## SNS Topic

### ImageModerationAlerts Topic

**Subscriptions:**
- Admin email (verified in SES)
- User email (verified in SES)

**Message Format:**
```json
{
  "articleId": "article-123",
  "imageKey": "uploads/image.jpg",
  "ownerId": "user-456",
  "violationType": "explicit-content",
  "severity": "high",
  "action": "deleted",
  "timestamp": "2025-12-08T10:30:00Z"
}
```

## Email Templates

### Admin Notification Email

![Email thông báo cho Admin](/images/5-Workshop/5.4-image-processing/Mail_SNS_FOR_ADMIN.png)

```
Subject: ⚠️ Content Moderation Alert - Image Violation Detected

Dear Admin,

An image has been flagged for content policy violation:

Article ID: {articleId}
Image: {imageKey}
Owner: {ownerId}
Violation Type: {violationType}
Severity: {severity}
Action Taken: {action}

Please review this case in the admin dashboard.

---
Travel Guide Moderation System
```

### User Notification Email
```
Subject: Your image was removed - Content Policy Violation

Hello,

We detected that one of your uploaded images violates our content policy:

Article ID: {articleId}
Violation Type: {violationType}
Action: Image has been removed

Please review our content guidelines and ensure future uploads comply with our policies.

If you believe this was a mistake, please contact support.

---
Travel Guide Team
```

## Demo Flow

### 1. Upload ảnh vi phạm
- Người dùng upload ảnh chứa nội dung không phù hợp
- S3 → SQS → Lambda Content Moderation

### 2. Phát hiện vi phạm
- Rekognition detect_moderation_labels trả về violation
- Lambda xác định mức độ nghiêm trọng

### 3. Publish SNS
```python
def send_admin_notification(article_id, key, moderation_result, owner_id):
    message = {
        'articleId': article_id,
        'imageKey': key,
        'ownerId': owner_id,
        'violationType': moderation_result['labels'],
        'severity': moderation_result['maxSeverity'],
        'action': 'deleted',
        'timestamp': datetime.utcnow().isoformat()
    }
    
    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Subject='⚠️ Content Moderation Alert',
        Message=json.dumps(message)
    )
```

### 4. SES gửi email
- Admin nhận email cảnh báo
- User nhận email thông báo ảnh bị xóa

## Kết quả Demo

✅ **Email Admin:**
- Nhận được thông báo về ảnh vi phạm
- Có đầy đủ thông tin để review
- Có thể xem chi tiết trong dashboard

✅ **Email User:**
- Được thông báo ảnh bị xóa
- Hiểu lý do vi phạm
- Biết cách liên hệ support nếu cần

## Monitoring

Theo dõi các metrics:
- **SNS NumberOfMessagesPublished** - Số message gửi đi
- **SES Send** - Số email gửi thành công
- **SES Bounce** - Email bị bounce
- **SES Complaint** - Email bị đánh dấu spam

## Kết luận

Hệ thống thông báo tự động:
- Đảm bảo admin biết ngay khi có vi phạm
- Người dùng được thông báo minh bạch
- Giảm thiểu công việc thủ công
- Tăng tính chuyên nghiệp của hệ thống
