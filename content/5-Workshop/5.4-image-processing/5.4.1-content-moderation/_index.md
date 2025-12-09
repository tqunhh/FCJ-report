---
title: "Lambda Content Moderation"
date: 2025-12-08
weight: 1
chapter: false
pre: "<b>5.4.1. </b>"
---

## Purpose

Automatically moderate image content to detect violations such as:
- Explicit/suggestive content
- Violence
- Drugs/alcohol
- Offensive symbols

## Main Code Explanation

### a. Receive message from SQS

Extract bucket and key information from S3 event:

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

### b. Call Rekognition detect_moderation_labels

AWS Rekognition analyzes the image and returns violation labels:

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

### c. Handle violations - Publish SNS to trigger SES email

If content violates policies, send admin notification:

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

## Demo: Email Notification for Violent Content

When violent content is detected, the system automatically sends email notifications:

![Email Notification - Violent Content](/images/5-Workshop/5.4-image-processing/Mail_Violent.png)

## Processing Flow

1. Image uploaded to S3
2. S3 event → SQS Queue
3. Lambda reads from SQS
4. Rekognition analyzes content
5. If passed → Forward to Detect Labels queue
6. If failed → Send notification + Stop pipeline

## Result

- Approved images continue to label detection
- Violated images are quarantined/deleted
- Admin and user receive email notifications
