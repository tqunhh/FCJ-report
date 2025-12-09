---
title: "CloudFront, S3 Upload & Location Service"
date: 2025-12-09
weight: 6
chapter: true
pre: "<b>5.6. </b>"
alwaysopen: false
---

## CloudFront (Static + Dynamic), S3 Upload Web, AWS Location Service

This section covers the implementation of **Amazon CloudFront** for content delivery, **Amazon S3** for image uploads using pre-signed URLs, and **AWS Location Service** for geocoding and mapping in the Travel Guide Application.

### Overview

The Travel Guide application uses three key AWS services to deliver a fast, secure, and feature-rich experience:

**Amazon CloudFront:**
- CDN for static web content (React build)
- CDN for images (articles, thumbnails, avatars, covers)
- HTTPS redirect and compression
- Origin Access Identity (OAI) for secure S3 access

**Amazon S3 Upload Web:**
- Pre-signed URLs for secure, direct uploads
- No AWS credentials needed on client
- Time-limited upload permissions (15 minutes)
- CORS configuration for browser uploads

**AWS Location Service:**
- Place Index for geocoding/reverse geocoding
- Esri data source
- DynamoDB cache for cost optimization
- Nominatim fallback for reliability

### Architecture

The system operates in three main flows:

#### Flow A — Static Web
1. User accesses React web application
2. CloudFront serves as CDN
3. CloudFront fetches content from S3 StaticSiteBucket (private) via OAI
4. Users always redirected to HTTPS with default index.html

#### Flow B — Image Upload & Display
1. Frontend calls `/upload-url` API to get pre-signed URL
2. Frontend uploads image directly to S3 ArticleImagesBucket
3. When displaying articles/images, frontend uses CloudFront domain for fast loading
4. CloudFront maps image paths to S3 origin

#### Flow C — Map/Geocoding
1. When creating/updating articles with coordinates, backend uses AWS Location Service Place Index
2. **Reverse geocoding**: lat/lng → place name
3. **Forward geocoding**: text → lat/lng (for search)
4. DynamoDB cache reduces Location Service costs
5. Nominatim fallback for reliability

### Content

- [S3 Pre-signed URL Upload](5.6.1-s3-presigned-upload/)
- [CloudFront CDN Configuration](5.6.2-cloudfront-cdn/)
- [AWS Location Service Integration](5.6.3-location-service/)

