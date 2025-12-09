---
title: "Input Sanitization"
date: 2025-12-09
weight: 2
chapter: false
pre: "<b>5.7.2. </b>"
---

## Input Sanitization - Ngăn chặn XSS

### Input Sanitization là gì?

**Input Sanitization** là quá trình làm sạch và xác thực user input để ngăn chặn code injection.

**Các tấn công được ngăn chặn:**
- **XSS (Cross-Site Scripting)** - Inject JavaScript
- **SQL/NoSQL Injection** - Thao túng database queries
- **Path Traversal** - Truy cập files trái phép
- **Command Injection** - Thực thi system commands

---

## Triển khai

### Security Utils Module

**File:** `security_utils.py`

**9 Functions:**
1. `sanitize_html()` - Escape HTML
2. `sanitize_string()` - Làm sạch strings
3. `validate_coordinates()` - Validate lat/lng
4. `validate_email()` - Validate email
5. `validate_s3_key()` - Ngăn path traversal
6. `validate_article_ownership()` - Kiểm tra ownership
7. `sanitize_tags()` - Validate tags
8. `validate_image_key()` - Validate image ownership
9. `rate_limit_key()` - Rate limiting

---

## Ví dụ

**TRƯỚC (Dễ bị tấn công):**
```python
title = data.get("title", "").strip()
# ❌ Không sanitization → XSS vulnerable
```

**SAU (An toàn):**
```python
from security_utils import sanitize_html

title = sanitize_html(data.get("title", ""))
# ✅ HTML escaped → XSS protected
```

---

## Validation Rules

### Title
- Max 200 ký tự
- HTML escaped
- Không control characters

### Content
- Max 10,000 ký tự
- HTML escaped

### Tags
- Max 10 tags
- Max 50 ký tự/tag

### Coordinates
- Latitude: -90 đến 90
- Longitude: -180 đến 180

### Images
- Max 10 MB
- Types: JPEG, PNG, GIF, WebP
- Ownership validated

---

## Testing

```bash
# Test XSS protection
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{\"title\":\"<script>alert(\\\"XSS\\\")</script>\"}'

# Expected: HTML escaped
```

---

## Điểm chính

1. Input sanitization ngăn XSS và injection attacks
2. HTML escaping là thiết yếu
3. Validation phải server-side
4. Security utils cung cấp functions tái sử dụng
5. Tất cả inputs phải được validate
