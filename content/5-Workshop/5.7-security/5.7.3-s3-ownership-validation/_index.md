---
title: "S3 Ownership Validation"
date: 2025-12-09
weight: 3
chapter: false
pre: "<b>5.7.3. </b>"
---

## S3 Ownership Validation - Prevent Unauthorized Access

### The Problem

**Before implementation:**
- Users could reference ANY S3 object in their articles
- No ownership validation
- Users could use other users' images
- Potential data leakage and storage abuse

**Attack scenario:**
```
1. User A uploads image: articles/user-a-id/photo.jpg
2. User B creates article with User A's image key
3. User B's article displays User A's private photo
4. ❌ Unauthorized access!
```

---

## The Solution

Implement **ownership validation** to ensure users can only use their own images.

**Validation checks:**
1. Image key format validation
2. Article ID matching
3. File size limits
4. File type validation
5. Ownership verification

---

## Implementation

### Updated: create_article.py

**Added validation logic:**

```python
from security_utils import validate_image_key
import boto3

s3_client = boto3.client('s3')
BUCKET_NAME = os.environ['ARTICLE_IMAGES_BUCKET']

def lambda_handler(event, context):
    data = json.loads(event['body'])
    user_id = event['requestContext']['authorizer']['claims']['sub']
    article_id = str(uuid.uuid4())
    
    # Get image keys from request
    image_keys = data.get("imageKeys", [])
    
    # ✅ Validate each image
    for key_str in image_keys:
        try:
            # 1. Validate key format and ownership
            validate_image_key(key_str, article_id, user_id)
            
            # 2. Check if image exists
            response = s3_client.head_object(
                Bucket=BUCKET_NAME,
                Key=key_str
            )
            
            # 3. Validate file size
            file_size = response['ContentLength']
            if file_size > 10 * 1024 * 1024:  # 10 MB
                return error(400, f"Image {key_str} is too large (max 10MB)")
            
            # 4. Validate file type
            content_type = response.get('ContentType', '')
            allowed_types = ['image/jpeg', 'image/png', 'image/gif', 'image/webp']
            
            if content_type not in allowed_types:
                return error(400, f"Invalid image type: {content_type}")
        
        except s3_client.exceptions.NoSuchKey:
            return error(404, f"Image not found: {key_str}")
        
        except PermissionError as e:
            return error(403, str(e))
        
        except Exception as e:
            return error(500, f"Error validating image: {str(e)}")
    
    # All images validated - proceed with article creation
    # ...
```

---

## Validation Functions

### 1. Key Format Validation

**Function:** `validate_image_key(key, article_id, owner_id)`

```python
def validate_image_key(key: str, article_id: str, owner_id: str) -> bool:
    """
    Validate that image key belongs to the article and user
    
    Expected format: articles/{article_id}/raw/{filename}
    or: articles/{article_id}/thumbnails/{filename}
    """
    # Basic S3 key validation
    validate_s3_key(key)
    
    # Check if key starts with correct article path
    expected_prefix = f"articles/{article_id}/"
    
    if not key.startswith(expected_prefix):
        raise PermissionError(
            f"Image does not belong to this article. "
            f"Expected prefix: {expected_prefix}"
        )
    
    # Additional checks for path traversal
    if '..' in key or '//' in key:
        raise PermissionError("Invalid image path detected")
    
    return True
```

### 2. File Size Validation

```python
def validate_file_size(file_size: int, max_size: int = 10 * 1024 * 1024) -> bool:
    """Validate file size (default max: 10 MB)"""
    if file_size > max_size:
        raise ValueError(f"File too large: {file_size} bytes (max: {max_size})")
    
    if file_size == 0:
        raise ValueError("File is empty")
    
    return True
```

### 3. File Type Validation

```python
def validate_file_type(content_type: str) -> bool:
    """Validate file MIME type"""
    allowed_types = [
        'image/jpeg',
        'image/jpg',
        'image/png',
        'image/gif',
        'image/webp'
    ]
    
    if content_type not in allowed_types:
        raise ValueError(
            f"Invalid file type: {content_type}. "
            f"Allowed: {', '.join(allowed_types)}"
        )
    
    return True
```

---

## S3 Object Metadata

### Storing Ownership Information

When uploading images, store metadata:

```python
def upload_image_with_metadata(file, article_id, user_id):
    """Upload image with ownership metadata"""
    s3_client.put_object(
        Bucket=BUCKET_NAME,
        Key=f"articles/{article_id}/raw/{filename}",
        Body=file,
        ContentType=content_type,
        Metadata={
            'article-id': article_id,
            'owner-id': user_id,
            'uploaded-at': datetime.now().isoformat()
        }
    )
```

![WAF Protection](/images/5-Workshop/5.7-security/WAF.jpg)

### Verifying Metadata

```python
def verify_image_ownership(key: str, user_id: str) -> bool:
    """Verify image ownership using S3 metadata"""
    try:
        response = s3_client.head_object(
            Bucket=BUCKET_NAME,
            Key=key
        )
        
        metadata = response.get('Metadata', {})
        owner = metadata.get('owner-id')
        
        if owner and owner != user_id:
            raise PermissionError("You don't own this image")
        
        return True
    
    except s3_client.exceptions.NoSuchKey:
        raise FileNotFoundError(f"Image not found: {key}")
```

---

## Testing

### Test 1: Valid Image (Own Article)

```bash
# Upload image first
curl -X POST https://api.example.com/upload-url \
  -H "Authorization: Bearer $TOKEN"

# Response: { "uploadUrl": "...", "key": "articles/abc123/raw/image.jpg", "articleId": "abc123" }

# Upload image to S3
curl -X PUT "$uploadUrl" \
  --upload-file image.jpg

# Create article with own image
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "My Article",
    "content": "Content",
    "latitude": 10,
    "longitude": 106,
    "imageKeys": ["articles/abc123/raw/image.jpg"]
  }'

# Expected: 200 OK - Article created
```

<!-- Image: Valid Image Test Success -->

### Test 2: Invalid Image (Other User's Image)

```bash
# Try to use another user's image
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Stolen Image",
    "content": "Content",
    "latitude": 10,
    "longitude": 106,
    "imageKeys": ["articles/other-user-id/raw/image.jpg"]
  }'

# Expected: 403 Forbidden
# Error: "Image does not belong to this article"
```

<!-- Image: Invalid Image Test Failure -->

### Test 3: File Too Large

```bash
# Try to upload 20MB image
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Large Image",
    "content": "Content",
    "latitude": 10,
    "longitude": 106,
    "imageKeys": ["articles/abc123/raw/large-image.jpg"]
  }'

# Expected: 400 Bad Request
# Error: "Image is too large (max 10MB)"
```

### Test 4: Invalid File Type

```bash
# Try to use PDF file
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "PDF File",
    "content": "Content",
    "latitude": 10,
    "longitude": 106,
    "imageKeys": ["articles/abc123/raw/document.pdf"]
  }'

# Expected: 400 Bad Request
# Error: "Invalid image type: application/pdf"
```

---

## Security Benefits

### 1. Prevent Unauthorized Access

✅ **Before:** Users could reference any S3 object
❌ **After:** Users can only use their own images

### 2. Prevent Data Leakage

✅ **Before:** Private images could be exposed
❌ **After:** Ownership validation prevents leakage

### 3. Prevent Storage Abuse

✅ **Before:** Users could link to unlimited images
❌ **After:** File size limits prevent abuse

### 4. Prevent Malware Uploads

✅ **Before:** Any file type accepted
❌ **After:** Only image types allowed

---

## Best Practices

### 1. Validate at Multiple Layers

✅ **Do:** Validate in Lambda AND S3 bucket policy
- Defense in depth
- Multiple checkpoints

### 2. Use S3 Metadata

✅ **Do:** Store ownership in metadata
- Easy to verify
- Immutable after upload

### 3. Implement Rate Limiting

✅ **Do:** Limit upload frequency
- Prevent abuse
- Reduce costs

```python
# Example rate limiting
from datetime import datetime, timedelta

def check_rate_limit(user_id: str) -> bool:
    """Check if user exceeded upload rate limit"""
    key = f"rate_limit:upload:{user_id}"
    
    # Get current count from cache
    count = cache.get(key) or 0
    
    # Limit: 10 uploads per hour
    if count >= 10:
        raise PermissionError("Upload rate limit exceeded")
    
    # Increment counter
    cache.set(key, count + 1, ex=3600)  # 1 hour TTL
    
    return True
```

### 4. Log All Validation Failures

✅ **Do:** Log suspicious activity
- Track attack attempts
- Identify patterns
- Alert on anomalies

```python
import logging

logger = logging.getLogger()

def log_validation_failure(user_id: str, key: str, reason: str):
    """Log validation failure for security monitoring"""
    logger.warning(
        f"Validation failed - User: {user_id}, Key: {key}, Reason: {reason}",
        extra={
            'user_id': user_id,
            'image_key': key,
            'failure_reason': reason,
            'timestamp': datetime.now().isoformat()
        }
    )
```

---

## Monitoring

### CloudWatch Metrics

**Track:**
- Validation failures per user
- Invalid image attempts
- File size violations
- File type violations

**Create alarms:**
```bash
# Alert on high validation failure rate
aws cloudwatch put-metric-alarm \
  --alarm-name high-validation-failures \
  --metric-name ValidationFailures \
  --namespace TravelGuide/Security \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold
```

### CloudTrail Logging

**Monitor S3 access:**
- Track who accessed what
- Identify unauthorized attempts
- Audit trail for compliance

---

## Cost Impact

### Storage Costs

**Before:**
- Users could link unlimited images
- No size limits
- Potential abuse

**After:**
- 10 MB per image limit
- Controlled storage growth
- Predictable costs

### API Costs

**Additional S3 API calls:**
- `head_object` per image validation
- ~$0.0004 per 1,000 requests
- **Negligible cost increase**

---

## Key Takeaways

1. **Ownership validation** prevents unauthorized image access
2. **File size limits** prevent storage abuse
3. **File type validation** prevents malware uploads
4. **S3 metadata** stores ownership information
5. **Multiple validation layers** provide defense in depth
6. **Logging and monitoring** detect attack attempts

