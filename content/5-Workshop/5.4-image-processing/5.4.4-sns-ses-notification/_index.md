---
title: "SNS & SES Notification"
date: 2025-12-08
weight: 4
chapter: false
pre: "<b>5.4.4. </b>"
---

## Purpose

Send automatic email notifications when:
- Content policy violations are detected
- Admin needs to be notified about violations
- Users need to know their images were removed/quarantined

## SNS & SES Architecture

```
Lambda Content Moderation
    ↓ (if violation detected)
SNS Topic: ImageModerationAlerts
    ↓
    ├─→ SES: Email to Admin
    └─→ SES: Email to User
```

## Demo: SNS Topic Configuration

SNS Topic is configured to distribute notifications to multiple subscribers:

![SNS Topic Configuration](/images/5-Workshop/5.4-image-processing/SNS.png)

## Demo: SES Verified Emails

Before sending emails, addresses must be verified in SES:

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

![Admin Email Notification](/images/5-Workshop/5.4-image-processing/Mail_SNS_FOR_ADMIN.png)

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

### 1. Upload violating image
- User uploads inappropriate content
- S3 → SQS → Lambda Content Moderation

### 2. Detect violation
- Rekognition detect_moderation_labels returns violation
- Lambda determines severity level

### 3. Publish to SNS
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

### 4. SES sends emails
- Admin receives alert email
- User receives notification about removed image

## Demo Results

✅ **Admin Email:**
- Receives violation notification
- Has complete information for review
- Can view details in dashboard

✅ **User Email:**
- Notified about image removal
- Understands violation reason
- Knows how to contact support if needed

## Monitoring

Track important metrics:
- **SNS NumberOfMessagesPublished** - Messages sent
- **SES Send** - Emails sent successfully
- **SES Bounce** - Bounced emails
- **SES Complaint** - Emails marked as spam

## Conclusion

Automated notification system:
- Ensures admin is immediately notified of violations
- Users are transparently informed
- Reduces manual work
- Increases system professionalism
