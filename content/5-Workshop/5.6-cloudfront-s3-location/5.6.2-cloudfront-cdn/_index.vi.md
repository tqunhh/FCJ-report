---
title: "Cấu hình CloudFront CDN"
date: 2025-12-09
weight: 2
chapter: false
pre: "<b>5.6.2. </b>"
---

## Amazon CloudFront (Static + Dynamic)

Trong dự án Travel Guide, CloudFront đóng vai trò CDN cho:
- **Web tĩnh** (React build) từ `StaticSiteBucket`
- **Ảnh** từ `ArticleImagesBucket` với path patterns

Phần "dynamic" (API) hiện đang chạy trực tiếp qua API Gateway URL. Workshop này mô tả **(1) thiết lập static đã triển khai** + **(2) mô hình dynamic khuyến nghị nếu muốn route API qua CloudFront**.

---

## Tổng quan Kiến trúc

![Kiến trúc CloudFront](/images/5-Workshop/5.6-cloudfront-s3-location/Architecture.jpg)

Hệ thống sử dụng CloudFront làm điểm truy cập thống nhất cho cả nội dung tĩnh và media files, cung cấp:
- Phân phối nội dung toàn cầu
- Bắt buộc HTTPS
- Nén dữ liệu
- Tối ưu caching

---

## CloudFront Static — Host React từ S3 Private

### Origin 1: S3 StaticSiteBucket (Private)

**Cấu hình:**
- Truy cập qua **Origin Access Identity (OAI)**
- `DefaultRootObject: index.html`
- `ViewerProtocolPolicy: redirect-to-https`
- Compression bật

![Tổng quan CloudFront Distribution](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Distribution_Overview.png)

### Thiết lập Origin Access Identity (OAI)

OAI cho phép CloudFront truy cập S3 buckets private một cách an toàn mà không cần public.

![Thiết lập CloudFront OAI](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_OAI_Setup.png)

**S3 Bucket Policy cho OAI:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontOAI",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity <OAI-ID>"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::static-site-bucket/*"
    }
  ]
}
```

---

## Triển khai Frontend

### Build và Upload

```bash
cd travel-guide-frontend
npm install
npm run build

# Upload build/ lên StaticSiteBucket
aws s3 sync build/ s3://<StaticSiteBucketName>/ --delete
```

### Invalidation sau khi Deploy

Sau mỗi lần deploy frontend, tạo invalidation để người dùng nhận phiên bản mới nhất:

```bash
aws cloudfront create-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --paths "/*"
```

![CloudFront Invalidation](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Invalidation.png)

**Best Practice:**
- Chỉ invalidate các path đã thay đổi để tiết kiệm chi phí
- Dùng versioned filenames (vd: `app.v123.js`) để tránh invalidation
- Invalidate `index.html` và các file không có hash

---

## CloudFront cho Ảnh — Cache và Tăng tốc Media Loading

### Origin 2: S3 ArticleImagesBucket (Private)

**Cache behaviors theo path:**
- `articles/*`
- `thumbnails/*`
- `avatars/*`
- `covers/*`

![Cấu hình CloudFront Origins](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Origins_Configuration.png)

### Cấu hình Cache Behaviors

![CloudFront Cache Behaviors](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Cache_Behaviors.png)

**Khi frontend cần hiển thị ảnh:**
- Tạo URL dạng: `https://<cloudfront-domain>/<key>`
- Ví dụ: `https://d1zx7zcxy3hbwd.cloudfront.net/articles/...`

**Lợi ích:**
- Ảnh được cache tại edge locations toàn cầu
- Giảm S3 GET requests
- Thời gian load nhanh hơn cho người dùng
- Tự động HTTPS

---

## Chiến lược Cache Khuyến nghị

### Web Tĩnh

**index.html:**
- TTL: Thấp (0-60 giây) hoặc dùng custom cache-control
- Lý do: Đảm bảo người dùng nhận phiên bản app mới nhất

**Hashed assets (js/css):**
- TTL: Cao (1 tuần - 1 năm)
- Lý do: Tên file thay đổi theo nội dung, an toàn cache lâu dài

### Ảnh

**Cấu hình:**
- TTL: Cao (ảnh hiếm khi thay đổi)
- Nếu ảnh thay đổi với cùng key → đổi key (best practice) hoặc invalidate

**TTL Khuyến nghị:**
- Min: 1 ngày
- Default: 7 ngày
- Max: 1 năm

---

## Giám sát và Metrics

### CloudWatch Metrics

![CloudFront Metrics](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Metrics.png)

**Metrics chính cần theo dõi:**

1. **Cache Hit Ratio** - Mục tiêu: > 80% cho static content
2. **4xx Error Rate** - Phổ biến: 403 (access denied), 404 (not found)
3. **5xx Error Rate** - Lỗi origin
4. **Bytes Downloaded** - Theo dõi bandwidth usage
5. **Requests** - Tổng requests theo thời gian

---

## Vận hành & Xử lý Sự cố

### Các vấn đề thường gặp

#### 1. 403 Access Denied
- Kiểm tra OAI ID trong bucket policy
- Xác minh object tồn tại trong S3

#### 2. Nội dung cũ sau khi Deploy
- Tạo CloudFront invalidation
- Dùng versioned filenames

#### 3. Load chậm lần đầu
- Hành vi bình thường (cold cache)
- Request đầu: Chậm (fetch từ origin)
- Requests tiếp theo: Nhanh (cached)

---

## Best Practices Bảo mật

1. **Chỉ HTTPS** - Redirect HTTP sang HTTPS
2. **Origin Access Identity** - Không bao giờ public S3 buckets
3. **Geo Restrictions** (Tùy chọn) - Whitelist/blacklist countries
4. **AWS WAF Integration** (Tùy chọn) - Bảo vệ chống web exploits

---

## Điểm chính

1. **CloudFront** tăng tốc phân phối nội dung toàn cầu
2. **OAI** bảo mật truy cập S3 private
3. **Cache behaviors** tối ưu các loại nội dung khác nhau
4. **Invalidation** đảm bảo người dùng nhận nội dung mới nhất
5. **TTL phù hợp** cân bằng giữa freshness và performance

