---
title: "Lambda Detect Labels"
date: 2025-12-08
weight: 2
chapter: false
pre: "<b>5.4.2. </b>"
---

## Mục đích

Phân tích ảnh và tự động gắn nhãn mô tả như:
- Địa danh (beach, mountain, city...)
- Đối tượng (people, food, building...)
- Hoạt động (swimming, hiking, dining...)

## Giải thích code chính

### a. Nhận message từ SQS

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

### b. Gọi rekognition.detect_labels

Phát hiện và ưu tiên các nhãn quan trọng:

```python
# Detect and prioritize labels
labels_data = detect_labels_in_image(bucket, key)

if not labels_data:
    print("No labels detected after prioritization")
    results['failed'] += 1
    continue
```

### c. Lưu labels vào ArticlesTable và Gallery

Cập nhật bài viết với tags và lưu vào Gallery:

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

## Demo: Dữ liệu sau khi xử lý

Sau khi xử lý thành công, dữ liệu được lưu vào các bảng DynamoDB:

![Dữ liệu sau khi xử lý](/images/5-Workshop/5.4-image-processing/DATA_AFTER_PROCESSING.png)

## Luồng xử lý

1. Nhận ảnh đã qua kiểm duyệt từ SQS
2. Gọi Rekognition detect_labels
3. Lọc và ưu tiên các nhãn quan trọng
4. Cập nhật ArticlesTable với tags
5. Lưu metadata vào GalleryPhotosTable
6. Cập nhật thống kê vào GalleryTrendsTable

## Kết quả

- Bài viết được gắn tags tự động
- Ảnh xuất hiện trong Gallery với tags
- Trending tags được cập nhật
- Người dùng có thể tìm kiếm theo tags
