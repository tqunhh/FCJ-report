---
title: "Encryption at Rest"
date: 2025-12-09
weight: 1
chapter: false
pre: "<b>5.7.1. </b>"
---

## Encryption at Rest - Data Protection

### What is Encryption at Rest?

**Encryption at Rest** means encrypting data when it's stored on disk (at rest), as opposed to data in transit (being transmitted over network).

**Why it matters:**
- Protects data if physical storage is compromised
- Compliance requirement (GDPR, HIPAA, PCI-DSS)
- Defense in depth security strategy
- Minimal performance impact with AWS managed encryption

---

## Implementation Overview

We implemented encryption for:
1. **DynamoDB Tables** (6 tables) - Using AWS KMS
2. **S3 Buckets** (2 buckets) - Using AES-256

---

## DynamoDB Encryption

### Tables Encrypted

All DynamoDB tables now use **AWS KMS encryption**:

1. ArticlesTable
2. UserFavoritesTable
3. GalleryPhotosTable
4. GalleryTrendsTable
5. UserProfilesTable
6. LocationCacheTable

### Configuration

**CloudFormation Template Changes:**

```yaml
ArticlesTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: !Sub "articles-${Environment}"
    # ... other properties ...
    
    # ✅ ADDED: KMS Encryption
    SSESpecification:
      SSEEnabled: true
      SSEType: KMS
```

![DynamoDB Encryption Settings](/images/5-Workshop/5.7-security/Dynamo_Encryption_Setting.jpg)

### How it Works

```
1. Application writes data to DynamoDB
   ↓
2. DynamoDB encrypts data using KMS key
   ↓
3. Encrypted data stored on disk
   ↓
4. When reading, DynamoDB decrypts automatically
   ↓
5. Application receives decrypted data
```

**Key Points:**
- Encryption/decryption is **automatic**
- No code changes required
- Transparent to application
- Uses AWS-managed KMS keys

### Benefits

✅ **Security:**
- Data encrypted at rest
- Protection against disk theft
- Compliance with regulations

✅ **Ease of Use:**
- Fully managed by AWS
- Automatic key rotation
- No key management burden

✅ **Performance:**
- Minimal latency impact (<1ms)
- No application changes needed

### Cost

**DynamoDB KMS Encryption:**
- KMS key: $1/month
- API calls: ~$0.03 per 10,000 requests
- **Estimated: $5-10/month** for typical usage

---

## S3 Encryption

### Buckets Encrypted

Both S3 buckets now use **AES-256 encryption**:

1. ArticleImagesBucket (user uploads)
2. StaticSiteBucket (frontend assets)

### Configuration

**CloudFormation Template Changes:**

```yaml
ArticleImagesBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "travel-guide-images-${Environment}-${AWS::AccountId}"
    # ... other properties ...
    
    # ✅ ADDED: AES-256 Encryption
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: AES256
          BucketKeyEnabled: true
```

![S3 Encryption Settings](/images/5-Workshop/5.7-security/S3_Encryption_Settings.jpg)

### How it Works

```
1. Application uploads file to S3
   ↓
2. S3 encrypts file using AES-256
   ↓
3. Encrypted file stored on disk
   ↓
4. When downloading, S3 decrypts automatically
   ↓
5. Application receives decrypted file
```

**Key Points:**
- Encryption is **server-side**
- Automatic for all objects
- No client-side changes needed
- Uses S3-managed keys (SSE-S3)

### Benefits

✅ **Security:**
- All files encrypted
- Protection against unauthorized access
- Compliance ready

✅ **Cost:**
- **FREE** - No additional cost for SSE-S3
- No performance impact

✅ **Simplicity:**
- Enabled at bucket level
- Applies to all objects
- No code changes

---

## Deployment

### Step 1: Update CloudFormation Template

**File:** `travel-guide-backend/core-infra/template.yaml`

**Changes made:**
- Added `SSESpecification` to all DynamoDB tables
- Added `BucketEncryption` to all S3 buckets

### Step 2: Deploy Stack

```bash
cd travel-guide-backend

# Deploy core infrastructure
./scripts/deploy-core.sh staging

# Or manually:
sam build -t core-infra/template.yaml
sam deploy \
  --stack-name travel-guide-core-staging \
  --parameter-overrides Environment=staging \
  --capabilities CAPABILITY_IAM
```

### Step 3: Verify Encryption

**Verify DynamoDB:**

```bash
# Check table encryption
aws dynamodb describe-table \
  --table-name articles-staging \
  --query 'Table.SSEDescription'

# Expected output:
{
  "Status": "ENABLED",
  "SSEType": "KMS"
}
```

**Verify S3:**

```bash
# Check bucket encryption
aws s3api get-bucket-encryption \
  --bucket travel-guide-images-staging-123456789012

# Expected output:
{
  "ServerSideEncryptionConfiguration": {
    "Rules": [
      {
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "AES256"
        },
        "BucketKeyEnabled": true
      }
    ]
  }
}
```

---

## Testing

### Test DynamoDB Encryption

```bash
# Create test article
curl -X POST https://api.example.com/articles \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Test Encryption",
    "content": "Sensitive data",
    "latitude": 10.8231,
    "longitude": 106.6297
  }'

# Verify data is encrypted in DynamoDB
# (You cannot see encrypted data directly - it's transparent)

# Read article back
curl https://api.example.com/articles/{article-id} \
  -H "Authorization: Bearer $TOKEN"

# Data should be readable (decrypted automatically)
```

### Test S3 Encryption

```bash
# Upload test image
aws s3 cp test-image.jpg \
  s3://travel-guide-images-staging-123456789012/test/

# Check object encryption
aws s3api head-object \
  --bucket travel-guide-images-staging-123456789012 \
  --key test/test-image.jpg \
  --query 'ServerSideEncryption'

# Expected: "AES256"
```

---

## Compliance

### Regulations Supported

✅ **GDPR (EU)**
- Article 32: Security of processing
- Encryption at rest required

✅ **HIPAA (Healthcare)**
- Technical safeguards
- Encryption required for PHI

✅ **PCI-DSS (Payment)**
- Requirement 3.4: Encryption of cardholder data
- Encryption at rest mandatory

✅ **SOC 2**
- Security principle
- Encryption controls

---

## Monitoring

### CloudWatch Metrics

**DynamoDB:**
- Monitor KMS API calls
- Track encryption errors
- Alert on decryption failures

**S3:**
- Monitor encryption status
- Track unencrypted uploads (should be 0)

### CloudTrail Logging

Enable CloudTrail to audit:
- KMS key usage
- S3 encryption changes
- Access to encrypted data

```bash
# Check CloudTrail logs
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=ResourceType,AttributeValue=AWS::KMS::Key \
  --max-results 10
```

---

## Best Practices

### 1. Use AWS-Managed Keys

✅ **Recommended:** AWS-managed KMS keys
- Automatic rotation
- No key management
- Lower cost

![KMS Key Management](/images/5-Workshop/5.7-security/KMS_Management.jpg)

![KMS Key Rotation](/images/5-Workshop/5.7-security/KMS_Rotation.jpg)

❌ **Avoid:** Customer-managed keys (unless required)
- Manual rotation
- Key management burden
- Higher cost

### 2. Enable Encryption by Default

✅ **Do:** Enable at bucket/table level
- Applies to all data
- Cannot be bypassed

❌ **Don't:** Rely on object-level encryption
- Easy to forget
- Inconsistent protection

### 3. Monitor Encryption Status

✅ **Do:** Set up CloudWatch alarms
- Alert on encryption disabled
- Track unencrypted objects

### 4. Document Encryption Keys

✅ **Do:** Document which keys encrypt what
- KMS key IDs
- Key policies
- Rotation schedules

---

## Troubleshooting

### Issue 1: KMS Access Denied

**Error:**
```
AccessDeniedException: User is not authorized to perform: kms:Decrypt
```

**Solution:**
- Add KMS permissions to Lambda execution role
- Grant `kms:Decrypt` and `kms:DescribeKey`

```json
{
  "Effect": "Allow",
  "Action": [
    "kms:Decrypt",
    "kms:DescribeKey"
  ],
  "Resource": "arn:aws:kms:*:*:key/*"
}
```

### Issue 2: S3 Encryption Not Applied

**Problem:** Old objects not encrypted

**Solution:**
- Encryption only applies to new objects
- Re-upload existing objects:

```bash
# Copy object to itself (re-encrypts)
aws s3 cp \
  s3://bucket/key \
  s3://bucket/key \
  --metadata-directive REPLACE
```

### Issue 3: Performance Impact

**Problem:** Slight latency increase

**Solution:**
- Normal with encryption (<1ms)
- Use S3 Bucket Keys to reduce KMS calls
- Already enabled: `BucketKeyEnabled: true`

---

## Cost Optimization

### DynamoDB KMS

**Reduce costs:**
1. Use AWS-managed keys (cheaper)
2. Enable S3 Bucket Keys (reduces KMS API calls)
3. Monitor KMS usage with Cost Explorer

### S3 AES-256

**No cost:**
- SSE-S3 (AES-256) is FREE
- No additional charges
- No performance impact

---

## Key Takeaways

1. **Encryption at rest** protects data on disk
2. **DynamoDB uses KMS** - automatic encryption/decryption
3. **S3 uses AES-256** - free and transparent
4. **No code changes** required - fully managed
5. **Compliance ready** - meets GDPR, HIPAA, PCI-DSS
6. **Minimal cost** - $5-10/month for DynamoDB KMS

