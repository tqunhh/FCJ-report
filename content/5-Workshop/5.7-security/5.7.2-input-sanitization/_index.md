---
title: "Input Sanitization"
date: 2025-12-09
weight: 2
chapter: false
pre: "<b>5.7.2. </b>"
---

## Input Sanitization - XSS Prevention

### What is Input Sanitization?

**Input Sanitization** is the process of cleaning and validating user input to prevent malicious code injection.

**Common attacks prevented:**
- **XSS (Cross-Site Scripting)** - Injecting JavaScript
- **SQL/NoSQL Injection** - Manipulating database queries
- **Path Traversal** - Accessing unauthorized files
- **Command Injection** - Executing system commands

---

## Implementation Overview

We created a comprehensive security utilities module with 9 functions to sanitize and validate all user inputs.

**File Created:** `travel-guide-backend/shared/layers/common/python/security_utils.py`

---

## Security Functions

### 1. HTML Sanitization

**Function:** `sanitize_html(text)`

**Purpose:** Escape HTML entities to prevent XSS attacks

**Before:**
```python
title = data.get("title", "").strip()
# ❌ Vulnerable: <script>alert('XSS')</script>
```

**After:**
```python
from security_utils import sanitize_html

title = sanitize_html(data.get("title", ""))
# ✅ Safe: &lt;script&gt;alert('XSS')&lt;/script&gt;
```

**Implementation:**
```python
import html

def sanitize_html(text: str) -> str:
    """Escape HTML entities to prevent XSS"""
    if not text:
        return ""
    
    # Escape HTML special characters
    sanitized = html.escape(text)
    
    # Remove null bytes
    sanitized = sanitized.replace('\x00', '')
    
    return sanitized.strip()
```

---

### 2. String Sanitization

**Function:** `sanitize_string(text, max_length=1000)`

**Purpose:** Clean and validate string inputs

```python
def sanitize_string(text: str, max_length: int = 1000) -> str:
    """Sanitize general string input"""
    if not text:
        return ""
    
    # Remove control characters
    sanitized = ''.join(char for char in text if char.isprintable() or char.isspace())
    
    # Trim whitespace
    sanitized = sanitized.strip()
    
    # Enforce max length
    if len(sanitized) > max_length:
        sanitized = sanitized[:max_length]
    
    return sanitized
```

---

### 3. Coordinate Validation

**Function:** `validate_coordinates(latitude, longitude)`

**Purpose:** Ensure coordinates are valid

```python
def validate_coordinates(latitude: float, longitude: float) -> tuple:
    """Validate geographic coordinates"""
    try:
        lat = float(latitude)
        lng = float(longitude)
        
        # Check ranges
        if not (-90 <= lat <= 90):
            raise ValueError("Latitude must be between -90 and 90")
        
        if not (-180 <= lng <= 180):
            raise ValueError("Longitude must be between -180 and 180")
        
        return (lat, lng)
    
    except (ValueError, TypeError) as e:
        raise ValueError(f"Invalid coordinates: {e}")
```

---

### 4. Email Validation

**Function:** `validate_email(email)`

**Purpose:** Validate email format

```python
import re

def validate_email(email: str) -> str:
    """Validate email address format"""
    if not email:
        raise ValueError("Email is required")
    
    # Basic email regex
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    
    if not re.match(pattern, email):
        raise ValueError("Invalid email format")
    
    return email.lower().strip()
```

---

### 5. S3 Key Validation

**Function:** `validate_s3_key(key)`

**Purpose:** Prevent path traversal attacks

```python
def validate_s3_key(key: str) -> str:
    """Validate S3 object key to prevent path traversal"""
    if not key:
        raise ValueError("S3 key is required")
    
    # Check for path traversal attempts
    if '..' in key or key.startswith('/'):
        raise ValueError("Invalid S3 key: path traversal detected")
    
    # Check for null bytes
    if '\x00' in key:
        raise ValueError("Invalid S3 key: null byte detected")
    
    return key.strip()
```

---

### 6. Article Ownership Validation

**Function:** `validate_article_ownership(article, user_id)`

**Purpose:** Ensure user owns the article

```python
def validate_article_ownership(article: dict, user_id: str) -> bool:
    """Validate that user owns the article"""
    if not article:
        raise ValueError("Article not found")
    
    article_owner = article.get('owner_id') or article.get('ownerId')
    
    if article_owner != user_id:
        raise PermissionError("You don't have permission to modify this article")
    
    return True
```

---

### 7. Tag Sanitization

**Function:** `sanitize_tags(tags)`

**Purpose:** Validate and limit tags

```python
def sanitize_tags(tags: list) -> list:
    """Sanitize and validate tags"""
    if not tags:
        return []
    
    # Limit number of tags
    MAX_TAGS = 10
    if len(tags) > MAX_TAGS:
        tags = tags[:MAX_TAGS]
    
    # Sanitize each tag
    sanitized_tags = []
    for tag in tags:
        if isinstance(tag, str):
            # Clean tag
            clean_tag = sanitize_string(tag, max_length=50)
            if clean_tag:
                sanitized_tags.append(clean_tag)
    
    return sanitized_tags
```

---

### 8. Image Key Validation

**Function:** `validate_image_key(key, article_id, owner_id)`

**Purpose:** Validate image ownership

```python
def validate_image_key(key: str, article_id: str, owner_id: str) -> bool:
    """Validate that image belongs to article and user"""
    # Check S3 key format
    validate_s3_key(key)
    
    # Check if key starts with correct article path
    expected_prefix = f"articles/{article_id}/"
    
    if not key.startswith(expected_prefix):
        raise PermissionError("Image does not belong to this article")
    
    return True
```

---

### 9. Rate Limiting Key

**Function:** `rate_limit_key(user_id, action)`

**Purpose:** Generate key for rate limiting

```python
def rate_limit_key(user_id: str, action: str) -> str:
    """Generate rate limit key for user action"""
    return f"rate_limit:{user_id}:{action}"
```

---

## Integration with Lambda Functions

### Updated: create_article.py

**Before (Vulnerable):**
```python
def lambda_handler(event, context):
    data = json.loads(event['body'])
    
    # ❌ No sanitization
    title = data.get("title", "").strip()
    content = data.get("content", "").strip()
    tags = data.get("tags", [])
    
    # Store directly in DynamoDB
    # → XSS vulnerable!
```

**After (Secure):**
```python
from security_utils import (
    sanitize_html,
    sanitize_tags,
    validate_coordinates,
    validate_image_key
)

def lambda_handler(event, context):
    data = json.loads(event['body'])
    user_id = event['requestContext']['authorizer']['claims']['sub']
    
    # ✅ Sanitize all inputs
    title = sanitize_html(data.get("title", ""))
    content = sanitize_html(data.get("content", ""))
    tags = sanitize_tags(data.get("tags", []))
    
    # ✅ Validate coordinates
    lat, lng = validate_coordinates(
        data.get("latitude"),
        data.get("longitude")
    )
    
    # ✅ Validate image ownership
    for image_key in data.get("imageKeys", []):
        validate_image_key(image_key, article_id, user_id)
    
    # Now safe to store
```

---

## Testing

### Test XSS Protection

```bash
# Attempt XSS attack
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "<script>alert(\"XSS\")</script>",
    "content": "<img src=x onerror=alert(1)>",
    "latitude": 10.8231,
    "longitude": 106.6297
  }'

# Expected: HTML escaped
# Title: "&lt;script&gt;alert(\"XSS\")&lt;/script&gt;"
# Content: "&lt;img src=x onerror=alert(1)&gt;"
```

### Test Coordinate Validation

```bash
# Invalid coordinates
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Test",
    "content": "Test",
    "latitude": 999,
    "longitude": -999
  }'

# Expected: 400 Bad Request
# Error: "Invalid coordinates: Latitude must be between -90 and 90"
```

### Test Tag Limits

```bash
# Too many tags
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Test",
    "content": "Test",
    "latitude": 10,
    "longitude": 106,
    "tags": ["tag1", "tag2", ..., "tag20"]
  }'

# Expected: Only first 10 tags saved
```

---

## Validation Rules

### Title
- Max length: 200 characters
- HTML escaped
- No control characters

### Content
- Max length: 10,000 characters
- HTML escaped
- No control characters

### Tags
- Max count: 10 tags
- Max length per tag: 50 characters
- Sanitized strings

### Coordinates
- Latitude: -90 to 90
- Longitude: -180 to 180
- Must be numbers

### Images
- Max size: 10 MB
- Allowed types: JPEG, PNG, GIF, WebP
- Ownership validated

---

## Best Practices

### 1. Sanitize at Entry Point

✅ **Do:** Sanitize in Lambda handler
- Before processing
- Before storing in database

❌ **Don't:** Sanitize in frontend only
- Can be bypassed
- Not secure

### 2. Validate Everything

✅ **Do:** Validate all user inputs
- Type checking
- Range checking
- Format validation

### 3. Use Allowlists

✅ **Do:** Define allowed values
- Allowed file types
- Allowed characters
- Allowed ranges

❌ **Don't:** Use blocklists
- Easy to bypass
- Incomplete protection

### 4. Escape Output

✅ **Do:** Escape when displaying
- HTML escape in templates
- URL encode in links
- JSON encode in APIs

---

## Key Takeaways

1. **Input sanitization** prevents XSS and injection attacks
2. **HTML escaping** is essential for user-generated content
3. **Validation** should happen server-side
4. **Security utils** provide reusable sanitization functions
5. **All inputs** must be validated - never trust user data
6. **Defense in depth** - sanitize at multiple layers

