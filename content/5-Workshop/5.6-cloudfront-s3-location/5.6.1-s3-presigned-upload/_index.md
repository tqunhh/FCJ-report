---
title: "S3 Pre-signed URL Upload"
date: 2025-12-09
weight: 1
chapter: false
pre: "<b>5.6.1. </b>"
---

## Amazon S3 Upload Web (Pre-signed URL)

### Why Use Pre-signed URLs?

Instead of uploading images through the backend (consuming bandwidth/risking timeout), we use a two-step approach:

1. Frontend requests a **temporary upload URL** from backend
2. Frontend uploads directly to S3

**Benefits:**
- Fast, reduces backend load
- Time-limited URL (security)
- No AWS credentials needed on client
- Direct upload to S3

---

## S3 Bucket Configuration

### ArticleImagesBucket Setup

**Key configurations:**
- **Versioning**: Enabled
- **CORS**: Allows uploads from frontend
- **Access**: Private (accessed via CloudFront OAI)
- **Event notifications**: Triggers SQS for image processing

![S3 Bucket Overview](/images/5-Workshop/5.6-cloudfront-s3-location/S3_OverView.png)

### CORS Configuration

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

![CORS Configuration](/images/5-Workshop/5.6-cloudfront-s3-location/CROS_Config.png)

**Note:** In production, replace `"*"` with specific domains.

---

## API: Generate Pre-signed URL

### Lambda Function

**Function**: `GetUploadUrlFunction`  
**API Path**: `POST /upload-url`  
**Authentication**: Cognito JWT required

### Request Flow

1. Frontend calls `POST {API_BASE}/upload-url` with JWT token
2. Backend generates pre-signed URL
3. Backend returns:
   - `uploadUrl`: Temporary S3 upload URL
   - `key`: Object path in S3
   - `expiresIn`: 900 seconds (15 minutes)

### Response Example

```json
{
  "uploadUrl": "https://s3.ap-southeast-1.amazonaws.com/...",
  "key": "articles/<articleId>/raw/<uuid>.jpg",
  "articleId": "<uuid>",
  "expiresIn": 900
}
```

### Lambda Code Structure

```python
import boto3
import uuid
from datetime import datetime

s3_client = boto3.client('s3')
BUCKET_NAME = os.environ['ARTICLE_IMAGES_BUCKET']

def lambda_handler(event, context):
    # Get user info from Cognito
    user_id = event['requestContext']['authorizer']['claims']['sub']
    
    # Generate unique key
    article_id = str(uuid.uuid4())
    file_key = f"articles/{article_id}/raw/{uuid.uuid4()}.jpg"
    
    # Generate pre-signed URL
    upload_url = s3_client.generate_presigned_url(
        'put_object',
        Params={
            'Bucket': BUCKET_NAME,
            'Key': file_key,
            'ContentType': 'image/jpeg'
        },
        ExpiresIn=900  # 15 minutes
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

## Frontend Upload Implementation

### React Example

```javascript
// Step 1: Get pre-signed URL
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

// Step 2: Upload file directly to S3
const uploadImage = async (file) => {
  // Get upload URL
  const { uploadUrl, key, articleId } = await getUploadUrl();
  
  // Upload to S3
  const uploadResponse = await fetch(uploadUrl, {
    method: 'PUT',
    headers: {
      'Content-Type': file.type
    },
    body: file
  });
  
  if (uploadResponse.ok) {
    console.log('Upload successful!');
    return { key, articleId };
  } else {
    throw new Error('Upload failed');
  }
};

// Usage
const handleFileSelect = async (event) => {
  const file = event.target.files[0];
  try {
    const result = await uploadImage(file);
    console.log('Image uploaded:', result);
  } catch (error) {
    console.error('Upload error:', error);
  }
};
```

---

## IAM Permissions

### Lambda Execution Role

The Lambda function needs `s3:PutObject` permission:

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

**Note:** Frontend doesn't need AWS credentials because it uses pre-signed URLs.

---

## Operations & Troubleshooting

### Common Issues

#### 1. 403 AccessDenied on PUT

**Causes:**
- URL expired (> 15 minutes)
- Content-Type mismatch (if enforced in pre-signed URL)
- Bucket policy blocking access

**Solution:**
- Request new upload URL
- Ensure Content-Type matches
- Check bucket policy

#### 2. CORS Error

**Causes:**
- CORS not configured on bucket
- Wrong method/headers in request

**Solution:**
- Verify CORS configuration on ArticleImagesBucket
- Ensure frontend sends correct headers
- Check browser console for specific CORS error

#### 3. Object Uploaded but Can't Load

**Cause:**
- Bucket is private

**Solution:**
- Load images via CloudFront (OAI)
- Or generate pre-signed download URLs

![S3 Object After Upload](/images/5-Workshop/5.6-cloudfront-s3-location/S3_Object_After_Upload.png)

### Monitoring

**CloudWatch Metrics to track:**
- Number of pre-signed URL requests
- Upload success/failure rate
- Average upload time
- S3 PUT request count

---

## Security Best Practices

1. **Time-limited URLs**: Keep expiration short (15 minutes)
2. **Content-Type validation**: Enforce image types only
3. **File size limits**: Set max file size in API
4. **User authentication**: Always require JWT token
5. **Bucket encryption**: Enable S3 encryption at rest
6. **Logging**: Enable S3 access logs for audit

---

## Cost Optimization

**Strategies:**
- Use S3 Intelligent-Tiering for infrequently accessed images
- Enable S3 Transfer Acceleration for global users (optional)
- Set lifecycle policies to delete incomplete multipart uploads
- Monitor S3 storage costs with Cost Explorer

---

## Key Takeaways

1. **Pre-signed URLs** enable secure, direct uploads without credentials
2. **CORS configuration** is essential for browser uploads
3. **Time-limited URLs** enhance security
4. **CloudFront** should be used for serving images (not direct S3)
5. **IAM least privilege** - Lambda only needs PutObject permission

