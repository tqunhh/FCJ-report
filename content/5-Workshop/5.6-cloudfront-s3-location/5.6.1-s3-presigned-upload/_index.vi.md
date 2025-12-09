---
title: "S3 Pre-signed URL Upload"
date: 2025-12-09
weight: 1
chapter: false
pre: "<b>5.6.1. </b>"
---

## Amazon S3 Upload Web (Pre-signed URL)

### Tại sao dùng Pre-signed URLs?

Thay vì upload ảnh qua backend (tốn băng thông/rủi ro timeout), chúng ta dùng phương pháp hai bước:

1. Frontend yêu cầu **URL upload tạm thời** từ backend
2. Frontend upload trực tiếp lên S3

**Lợi ích:**
- Nhanh, giảm tải cho backend
- URL giới hạn thời gian (bảo mật)
- Không cần AWS credentials trên client
- Upload trực tiếp lên S3

---

## Cấu hình S3 Bucket

### Thiết lập ArticleImagesBucket

**Cấu hình chính:**
- **Versioning**: Bật
- **CORS**: Cho phép upload từ frontend
- **Access**: Private (truy cập qua CloudFront OAI)
- **Event notifications**: Trigger SQS để xử lý ảnh

![S3 Bucket Overview](/images/5-Workshop/5.6-cloudfront-s3-location/S3_OverView.png)

### Cấu hình CORS

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["ETag"]
  }
]
```

![Cấu hình CORS](/images/5-Workshop/5.6-cloudfront-s3-location/CROS_Config.png)

**Lưu ý:** Trong production, thay `"*"` bằng domain cụ thể.

---

## API: Tạo Pre-signed URL

### Lambda Function

**Function**: `GetUploadUrlFunction`  
**API Path**: `POST /upload-url`  
**Xác thực**: Yêu cầu Cognito JWT

### Luồng Request

1. Frontend gọi `POST {API_BASE}/upload-url` với JWT token
2. Backend tạo pre-signed URL
3. Backend trả về:
   - `uploadUrl`: URL upload S3 tạm thời
   - `key`: Đường dẫn object trong S3
   - `expiresIn`: 900 giây (15 phút)

### Ví dụ Response

```json
{
  "uploadUrl": "https://s3.ap-southeast-1.amazonaws.com/...",
  "key": "articles/<articleId>/raw/<uuid>.jpg",
  "articleId": "<uuid>",
  "expiresIn": 900
}
```

### Cấu trúc Lambda Code

```python
import boto3
import uuid
from datetime import datetime

s3_client = boto3.client('s3')
BUCKET_NAME = os.environ['ARTICLE_IMAGES_BUCKET']

def lambda_handler(event, context):
    # Lấy thông tin user từ Cognito
    user_id = event['requestContext']['authorizer']['claims']['sub']
    
    # Tạo key duy nhất
    article_id = str(uuid.uuid4())
    file_key = f"articles/{article_id}/raw/{uuid.uuid4()}.jpg"
    
    # Tạo pre-signed URL
    upload_url = s3_client.generate_presigned_url(
        'put_object',
        Params={
            'Bucket': BUCKET_NAME,
            'Key': file_key,
            'ContentType': 'image/jpeg'
        },
        ExpiresIn=900  # 15 phút
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'uploadUrl': upload_url,
            'key': file_key,
            'articleId': article_id,
            'expiresIn': 900
        })
    }
```

---

## Triển khai Upload trên Frontend

### Ví dụ React

```javascript
// Bước 1: Lấy pre-signed URL
const getUploadUrl = async () => {
  const response = await fetch(`${API_BASE}/upload-url`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    }
  });
  
  return await response.json();
};

// Bước 2: Upload file trực tiếp lên S3
const uploadImage = async (file) => {
  // Lấy upload URL
  const { uploadUrl, key, articleId } = await getUploadUrl();
  
  // Upload lên S3
  const uploadResponse = await fetch(uploadUrl, {
    method: 'PUT',
    headers: {
      'Content-Type': file.type
    },
    body: file
  });
  
  if (uploadResponse.ok) {
    console.log('Upload thành công!');
    return { key, articleId };
  } else {
    throw new Error('Upload thất bại');
  }
};

// Sử dụng
const handleFileSelect = async (event) => {
  const file = event.target.files[0];
  try {
    const result = await uploadImage(file);
    console.log('Ảnh đã upload:', result);
  } catch (error) {
    console.error('Lỗi upload:', error);
  }
};
```

---

## Quyền IAM

### Lambda Execution Role

Lambda function cần quyền `s3:PutObject`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl"
      ],
      "Resource": "arn:aws:s3:::article-images-bucket/*"
    }
  ]
}
```

**Lưu ý:** Frontend không cần AWS credentials vì dùng pre-signed URLs.

---

## Vận hành & Xử lý Sự cố

### Các vấn đề thường gặp

#### 1. 403 AccessDenied khi PUT

**Nguyên nhân:**
- URL hết hạn (> 15 phút)
- Content-Type không khớp (nếu bắt buộc trong pre-signed URL)
- Bucket policy chặn truy cập

**Giải pháp:**
- Yêu cầu upload URL mới
- Đảm bảo Content-Type khớp
- Kiểm tra bucket policy

#### 2. Lỗi CORS

**Nguyên nhân:**
- CORS chưa cấu hình trên bucket
- Method/headers sai trong request

**Giải pháp:**
- Xác minh cấu hình CORS trên ArticleImagesBucket
- Đảm bảo frontend gửi đúng headers
- Kiểm tra browser console để xem lỗi CORS cụ thể

#### 3. Object Upload xong nhưng không load được

**Nguyên nhân:**
- Bucket là private

**Giải pháp:**
- Load ảnh qua CloudFront (OAI)
- Hoặc tạo pre-signed download URLs

![S3 Object sau khi Upload](/images/5-Workshop/5.6-cloudfront-s3-location/S3_Object_After_Upload.png)

### Giám sát

**CloudWatch Metrics cần theo dõi:**
- Số lượng pre-signed URL requests
- Tỷ lệ upload thành công/thất bại
- Thời gian upload trung bình
- Số lượng S3 PUT requests

---

## Best Practices Bảo mật

1. **URL giới hạn thời gian**: Giữ thời gian hết hạn ngắn (15 phút)
2. **Xác thực Content-Type**: Chỉ cho phép loại ảnh
3. **Giới hạn kích thước file**: Đặt max file size trong API
4. **Xác thực người dùng**: Luôn yêu cầu JWT token
5. **Mã hóa bucket**: Bật S3 encryption at rest
6. **Logging**: Bật S3 access logs để audit

---

## Tối ưu Chi phí

**Chiến lược:**
- Dùng S3 Intelligent-Tiering cho ảnh ít truy cập
- Bật S3 Transfer Acceleration cho người dùng toàn cầu (tùy chọn)
- Đặt lifecycle policies để xóa incomplete multipart uploads
- Giám sát chi phí S3 storage với Cost Explorer

---

## Điểm chính

1. **Pre-signed URLs** cho phép upload an toàn trực tiếp không cần credentials
2. **Cấu hình CORS** là thiết yếu cho browser uploads
3. **URL giới hạn thời gian** tăng cường bảo mật
4. **CloudFront** nên được dùng để serve ảnh (không phải S3 trực tiếp)
5. **IAM least privilege** - Lambda chỉ cần quyền PutObject

