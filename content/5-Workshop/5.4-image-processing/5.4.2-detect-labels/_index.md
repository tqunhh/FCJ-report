---
title: "Lambda Detect Labels"
date: 2025-12-08
weight: 2
chapter: false
pre: "<b>5.4.2. </b>"
---

## Purpose

Analyze images and automatically add descriptive labels such as:
- Locations (beach, mountain, city...)
- Objects (people, food, building...)
- Activities (swimming, hiking, dining...)

## Main Code Explanation

### a. Receive message from SQS

```python
for sqs_record in event.get('Records', []):
    try:
        # Parse S3 event from SQS message body
        s3_event = json.loads(sqs_record['body'])
        
        # Process each S3 record in the event
        for s3_record in s3_event.get('Records', []):
            try:
                # Extract S3 information
                bucket = s3_record['s3']['bucket']['name']
                key = s3_record['s3']['object']['key']
```

### b. Call rekognition.detect_labels

Detect and prioritize important labels:

```python
# Detect and prioritize labels
labels_data = detect_labels_in_image(bucket, key)

if not labels_data:
    print("No labels detected after prioritization")
    results['failed'] += 1
    continue
```

### c. Save labels to ArticlesTable and Gallery

Update article with tags and save to Gallery:

```python
# Update article
success = update_article_with_tags(article_id, labels_data)

if success:
    results['succeeded'] += 1
    
    # Save to Gallery tables
    try:
        import sys
        import os
        # Add current directory to path
        current_dir = os.path.dirname(os.path.abspath(__file__))
        if current_dir not in sys.path:
            sys.path.insert(0, current_dir)
        
        from save_to_gallery import save_photo_to_gallery, update_trending_tags
        
        tag_names = [label['name'] for label in labels_data]
        image_url = key  # S3 key
        photo_id = key  # Use S3 key as unique identifier
        
        save_photo_to_gallery(
            photo_id, 
            image_url, 
            tag_names, 
            status='public', 
            article_id=article_id
        )
        update_trending_tags(tag_names, image_url)
        print("✓ Saved to Gallery tables")
```

## Demo: Data After Processing

After successful processing, data is stored in DynamoDB tables:

![Data After Processing](/images/5-Workshop/5.4-image-processing/DATA_AFTER_PROCESSING.png)

## Processing Flow

1. Receive moderated image from SQS
2. Call Rekognition detect_labels
3. Filter and prioritize important labels
4. Update ArticlesTable with tags
5. Save metadata to GalleryPhotosTable
6. Update statistics in GalleryTrendsTable

## Result

- Articles are automatically tagged
- Images appear in Gallery with tags
- Trending tags are updated
- Users can search by tags
