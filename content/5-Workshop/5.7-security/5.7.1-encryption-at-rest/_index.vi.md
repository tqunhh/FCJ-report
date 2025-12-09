---
title: "Encryption at Rest"
date: 2025-12-09
weight: 1
chapter: false
pre: "<b>5.7.1. </b>"
---

## Encryption at Rest - Bảo vệ Dữ liệu

### Encryption at Rest là gì?

**Encryption at Rest** có nghĩa là mã hóa dữ liệu khi được lưu trữ trên đĩa, khác với dữ liệu đang truyền tải qua mạng.

**Tại sao quan trọng:**
- Bảo vệ dữ liệu nếu storage vật lý bị xâm phạm
- Yêu cầu tuân thủ (GDPR, HIPAA, PCI-DSS)
- Chiến lược bảo mật nhiều lớp
- Tác động performance tối thiểu

---

## Triển khai

### DynamoDB Tables (6 tables)
- Sử dụng AWS KMS encryption
- Tự động encryption/decryption
- Chi phí: ~$5-10/tháng

### S3 Buckets (2 buckets)
- Sử dụng AES-256 encryption
- MIỄN PHÍ
- Không ảnh hưởng performance

---

## Cấu hình

**DynamoDB:**
```yaml
SSESpecification:
  SSEEnabled: true
  SSEType: KMS
```

**S3:**
```yaml
BucketEncryption:
  ServerSideEncryptionConfiguration:
    - ServerSideEncryptionByDefault:
        SSEAlgorithm: AES256
      BucketKeyEnabled: true
```

---

## Xác minh

**DynamoDB:**
```bash
aws dynamodb describe-table --table-name articles-staging --query 'Table.SSEDescription'
```

**S3:**
```bash
aws s3api get-bucket-encryption --bucket travel-guide-images-staging-123456789012
```

---

## Điểm chính

1. Encryption at rest bảo vệ dữ liệu trên đĩa
2. DynamoDB dùng KMS - tự động
3. S3 dùng AES-256 - miễn phí
4. Không cần thay đổi code
5. Tuân thủ GDPR, HIPAA, PCI-DSS
