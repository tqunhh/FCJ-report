---
title: "CloudFront CDN Configuration"
date: 2025-12-09
weight: 2
chapter: false
pre: "<b>5.6.2. </b>"
---

## Amazon CloudFront (Static + Dynamic)

In the Travel Guide project, CloudFront serves as the CDN for:
- **Static web** (React build) from `StaticSiteBucket`
- **Images** from `ArticleImagesBucket` with path patterns

The "dynamic" part (API) currently runs directly through API Gateway URL. This workshop describes **(1) deployed static setup** + **(2) recommended dynamic model if you want to route API through CloudFront**.

---

## Architecture Overview

![CloudFront Architecture](/images/5-Workshop/5.6-cloudfront-s3-location/Architecture.jpg)

The system uses CloudFront as a unified entry point for both static content and media files, providing:
- Global content delivery
- HTTPS enforcement
- Compression
- Caching optimization

---

## CloudFront Static — Host React from Private S3

### Origin 1: S3 StaticSiteBucket (Private)

**Configuration:**
- Access via **Origin Access Identity (OAI)**
- `DefaultRootObject: index.html`
- `ViewerProtocolPolicy: redirect-to-https`
- Compression enabled

![CloudFront Distribution Overview](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Distribution_Overview.png)

### Origin Access Identity (OAI) Setup

OAI allows CloudFront to access private S3 buckets securely without making them public.

![CloudFront OAI Setup](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_OAI_Setup.png)

**S3 Bucket Policy for OAI:**

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

## Deploying Frontend

### Build and Upload

```bash
cd travel-guide-frontend
npm install
npm run build

# Upload build/ to StaticSiteBucket
aws s3 sync build/ s3://<StaticSiteBucketName>/ --delete
```

### Invalidation After Deploy

After each frontend deployment, create an invalidation so users receive the latest version:

```bash
aws cloudfront create-invalidation \
  --distribution-id <DISTRIBUTION_ID> \
  --paths "/*"
```

![CloudFront Invalidation](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Invalidation.png)

**Best Practice:**
- Invalidate only changed paths to save costs
- Use versioned filenames (e.g., `app.v123.js`) to avoid invalidation
- Invalidate `index.html` and other non-hashed files

---

## CloudFront for Images — Cache and Accelerate Media Loading

### Origin 2: S3 ArticleImagesBucket (Private)

**Cache behaviors by path:**
- `articles/*`
- `thumbnails/*`
- `avatars/*`
- `covers/*`

![CloudFront Origins Configuration](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Origins_Configuration.png)

### Cache Behaviors Configuration

![CloudFront Cache Behaviors](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Cache_Behaviors.png)

**When frontend needs to display images:**
- Construct URL like: `https://<cloudfront-domain>/<key>`
- Example: `https://d1zx7zcxy3hbwd.cloudfront.net/articles/...`

**Benefits:**
- Images cached at edge locations globally
- Reduced S3 GET requests
- Faster load times for users
- Automatic HTTPS

---

## Cache Strategy Recommendations

### Static Web

**index.html:**
- TTL: Low (0-60 seconds) or use custom cache-control
- Reason: Ensure users get latest app version

**Hashed assets (js/css):**
- TTL: High (1 week - 1 year)
- Reason: Filenames change with content, safe to cache long-term

**Example Cache-Control Headers:**

```
# index.html
Cache-Control: no-cache, must-revalidate

# app.abc123.js
Cache-Control: public, max-age=31536000, immutable
```

### Images

**Configuration:**
- TTL: High (images rarely change)
- If image changes with same key → change key (best practice) or invalidate

**Recommended TTL:**
- Min: 1 day
- Default: 7 days
- Max: 1 year

---

## CloudFront Dynamic — Recommended Model (Optional)

If you want "dynamic" (API) to run through CloudFront, add:

### Origin 3: API Gateway Endpoint (Custom Origin)

**Configuration:**
- Behavior: `/api/*` (or `/Prod/*`) → API origin
- Cache: Disabled or very low TTL
- Forward headers: `Authorization` + query strings
- Forward cookies: As needed

**Goals:**
- Single domain for both web + API
- Reduce global latency for API (depending on use case)

**Note:** Configuring API through CloudFront requires proper path normalization and CORS setup. Also consider caching strategy to avoid "wrong cache" based on tokens.

---

## Monitoring and Metrics

### CloudWatch Metrics

![CloudFront Metrics](/images/5-Workshop/5.6-cloudfront-s3-location/CloudFront_Metrics.png)

**Key metrics to monitor:**

1. **Cache Hit Ratio**
   - Target: > 80% for static content
   - Low ratio indicates caching issues

2. **4xx Error Rate**
   - Common: 403 (access denied), 404 (not found)
   - Check OAI permissions and file paths

3. **5xx Error Rate**
   - Origin errors
   - Check S3 availability

4. **Bytes Downloaded**
   - Track bandwidth usage
   - Identify popular content

5. **Requests**
   - Total requests per time period
   - Identify traffic patterns

---

## Operations & Troubleshooting

### Common Issues

#### 1. 403 Access Denied

**Causes:**
- OAI not configured correctly
- S3 bucket policy missing CloudFront permission
- Object doesn't exist

**Solution:**
- Verify OAI ID in bucket policy
- Check object exists in S3
- Test direct S3 access (temporarily)

#### 2. Stale Content After Deploy

**Causes:**
- No invalidation created
- Browser cache

**Solution:**
- Create CloudFront invalidation
- Use versioned filenames
- Set appropriate Cache-Control headers

#### 3. Slow First Load

**Cause:**
- Cold cache (first request to edge location)

**Expected behavior:**
- First request: Slower (origin fetch)
- Subsequent requests: Fast (cached)

#### 4. High Origin Requests

**Causes:**
- Low cache hit ratio
- TTL too short
- Query strings not handled properly

**Solution:**
- Increase TTL for static content
- Configure query string forwarding
- Use cache key normalization

---

## Security Best Practices

1. **HTTPS Only**
   - Redirect HTTP to HTTPS
   - Use TLS 1.2 minimum

2. **Origin Access Identity**
   - Never make S3 buckets public
   - Use OAI for CloudFront access

3. **Geo Restrictions** (Optional)
   - Whitelist/blacklist countries if needed

4. **AWS WAF Integration** (Optional)
   - Protect against common web exploits
   - Rate limiting

5. **Signed URLs/Cookies** (Optional)
   - For premium/private content
   - Time-limited access

---

## Cost Optimization

**Strategies:**

1. **High Cache Hit Ratio**
   - Reduces origin requests
   - Lower S3 GET costs

2. **Compression**
   - Reduces data transfer
   - Faster load times

3. **Price Class Selection**
   - Use fewer edge locations if users are regional
   - Price Class 100 (US, Europe, Asia)

4. **Reserved Capacity** (High traffic)
   - Contact AWS for discounts

5. **Monitor Invalidations**
   - First 1,000 paths/month free
   - Use versioned filenames instead

---

## Key Takeaways

1. **CloudFront** accelerates content delivery globally
2. **OAI** secures private S3 access
3. **Cache behaviors** optimize different content types
4. **Invalidation** ensures users get latest content
5. **Monitoring** helps identify and resolve issues
6. **Proper TTL** balances freshness and performance

