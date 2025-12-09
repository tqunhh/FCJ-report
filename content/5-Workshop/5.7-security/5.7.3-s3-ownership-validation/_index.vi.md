---
title: "S3 Ownership Validation"
date: 2025-12-09
weight: 3
chapter: false
pre: "<b>5.7.3. </b>"
---

## S3 Ownership Validation - Ngăn Truy cập Trái phép

### Vấn đề

**Trước khi triển khai:**
- Users có thể reference BẤT KỲ S3 object nào
- Không có ownership validation
- Users có thể dùng ảnh của người khác
- Rò rỉ dữ liệu và lạm dụng storage

---

## Giải pháp

Triển khai **ownership validation** để đảm bảo users chỉ có thể dùng ảnh của mình.

**Validation checks:**
1. Image key format validation
2. Article ID matching
3. File size limits (10 MB)
4. File type validation
5. Ownership verification

---

## Triển khai

### Validation Logic

```python
from security_utils import validate_image_key

# Validate mỗi image
for key_str in image_keys:
    # 1. Validate format và ownership
    validate_image_key(key_str, article_id, user_id)
    
    # 2. Kiểm tra image tồn tại
    response = s3_client.head_object(Bucket=BUCKET_NAME, Key=key_str)
    
    # 3. Validate file size
    if response['ContentLength'] > 10 * 1024 * 1024:
        return error(400, "Image quá lớn (max 10MB)")
    
    # 4. Validate file type
    if response['ContentType'] not in allowed_types:
        return error(400, "Invalid image type")
```

---

## Validation Functions

### 1. Key Format Validation

```python
def validate_image_key(key, article_id, owner_id):
    # Kiểm tra key bắt đầu với đúng article path
    expected_prefix = f"articles/{article_id}/"
    
    if not key.startswith(expected_prefix):
        raise PermissionError("Image không thuộc article này")
    
    return True
```

### 2. File Size Validation

```python
def validate_file_size(file_size, max_size=10*1024*1024):
    if file_size > max_size:
        raise ValueError(f"File quá lớn: {file_size} bytes")
    return True
```

### 3. File Type Validation

```python
def validate_file_type(content_type):
    allowed = ['image/jpeg', 'image/png', 'image/gif', 'image/webp']
    if content_type not in allowed:
        raise ValueError(f"Invalid file type: {content_type}")
    return True
```

---

## Testing

### Test 1: Valid Image (Own Article)

```bash
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{\"imageKeys\":[\"articles/abc123/raw/image.jpg\"]}'

# Expected: 200 OK
```

### Test 2: Invalid Image (Other User's Image)

```bash
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{\"imageKeys\":[\"articles/other-user/raw/image.jpg\"]}'

# Expected: 403 Forbidden
```

### Test 3: File Too Large

```bash
# Expected: 400 Bad Request
# Error: "Image quá lớn (max 10MB)"
```

### Test 4: Invalid File Type

```bash
# Expected: 400 Bad Request
# Error: "Invalid image type"
```

---

## Lợi ích Bảo mật

### 1. Ngăn Truy cập Trái phép
 Users chỉ có thể dùng ảnh của mình

### 2. Ngăn Rò rỉ Dữ liệu
 Ảnh private không thể bị lộ

### 3. Ngăn Lạm dụng Storage
 File size limits ngăn abuse

### 4. Ngăn Malware Uploads
 Chỉ cho phép image types

---

## Best Practices

1. **Validate ở nhiều layers** - Lambda AND S3 bucket policy
2. **Dùng S3 Metadata** - Lưu ownership information
3. **Implement Rate Limiting** - Giới hạn upload frequency
4. **Log validation failures** - Theo dõi attack attempts

---

## Giám sát

### CloudWatch Metrics

**Theo dõi:**
- Validation failures per user
- Invalid image attempts
- File size violations
- File type violations

---

## Điểm chính

1. Ownership validation ngăn truy cập trái phép
2. File size limits ngăn storage abuse
3. File type validation ngăn malware uploads
4. S3 metadata lưu ownership information
5. Multiple validation layers cung cấp defense in depth
6. Logging và monitoring phát hiện attack attempts
