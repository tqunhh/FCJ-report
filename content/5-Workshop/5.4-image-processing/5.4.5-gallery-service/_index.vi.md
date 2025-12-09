---
title: "Gallery Service - Khám phá Ảnh"
date: 2025-12-09
weight: 5
chapter: false
pre: "<b>5.4.5. </b>"
---

## Tổng quan Gallery Service

Gallery Service cung cấp chức năng khám phá ảnh thông qua trending tags và tìm kiếm theo tags. Nó consume dữ liệu từ AI Service's label detection và trình bày qua các APIs thân thiện với người dùng.

### Mục đích và Use Cases

**Chức năng chính:**
1. **Trending Tags**: Hiển thị các tags phổ biến nhất dựa trên số lượng photos
2. **Photo Gallery**: Tìm kiếm photos theo tags cụ thể

**Vị trí trong Hệ thống:**
- Consume dữ liệu từ AI Service (auto-detected tags)
- Cung cấp API cho Frontend để hiển thị gallery
- Đọc từ DynamoDB tables được populate bởi AI Service

---

## Kiến trúc

### Tổng quan Components

![Gallery API Gateway](/images/5-Workshop/5.4-image-processing/Gallery_API_Gateway.png)

Gallery Service bao gồm:
- **2 Lambda Functions**: GetTrendingTags, GetArticlesByTag
- **1 API Gateway**: REST API với 2 endpoints
- **1 Lambda Layer**: Shared dependencies (boto3, CORS utilities)
- **2 DynamoDB Tables**: GalleryPhotosTable, GalleryTrendsTable (từ Core Stack)

### Luồng Dữ liệu

```
Người dùng Upload Ảnh
    ↓
Article Service → S3 Bucket
    ↓
S3 Event → AI Service
    ↓
Label Detection
    ↓
├─→ Lưu vào GalleryPhotosTable
└─→ Cập nhật GalleryTrendsTable (tăng count)
    ↓
Gallery Service (Chỉ đọc)
    ↓
Hiển thị Frontend
```

### Dependencies với Core Stack

Gallery Service import từ Core Stack:

```yaml
# Từ core-infra stack
Imports:
  - GalleryPhotosTableName
  - GalleryTrendsTableName
```

**Thứ tự Deployment:**
1. Core Stack (tạo tables)
2. AI Service (populate tables)
3. Gallery Service (đọc tables)

---

## Lambda Functions

### Function 1: GetTrendingTagsFunction

**Mục đích:** Trả về danh sách trending tags được sắp xếp theo độ phổ biến

**Cấu hình:**
- Runtime: Python 3.11
- Timeout: 30 giây
- Memory: 512 MB
- Handler: `get_trending_tags.lambda_handler`

![GetTrendingTags Function](/images/5-Workshop/5.4-image-processing/GetTrendingTags_Function.png)

**Luồng Logic:**

```
1. Parse query parameters (limit)
2. Scan tất cả tags từ GalleryTrendsTable
3. Sắp xếp theo count (giảm dần) trong memory
4. Trả về top N tags
```

**Cấu trúc Code:**

```python
def lambda_handler(event, context):
    # Parse parameters
    limit = int(params.get("limit", 20))
    
    # Scan tất cả tags
    all_tags = []
    while True:
        response = table.scan(**scan_kwargs)
        all_tags.extend(response.get('Items', []))
        if 'LastEvaluatedKey' not in response:
            break
    
    # Sắp xếp theo count
    all_tags.sort(key=lambda x: x.get('count', 0), reverse=True)
    
    # Trả về top N
    return {
        'statusCode': 200,
        'body': json.dumps({
            'items': all_tags[:limit],
            'total_tags': len(all_tags)
        })
    }
```

**Ví dụ Response:**

```json
{
  "items": [
    {
      "tag_name": "Beach",
      "count": 150,
      "cover_image": "articles/abc123/image.jpg",
      "last_updated": "2024-01-15T10:30:00Z"
    },
    {
      "tag_name": "Mountain",
      "count": 120,
      "cover_image": "articles/def456/image.jpg",
      "last_updated": "2024-01-14T15:20:00Z"
    }
  ],
  "total_tags": 500
}
```

---

### Function 2: GetArticlesByTagFunction

**Mục đích:** Tìm kiếm photos có tag cụ thể

**Cấu hình:**
- Runtime: Python 3.11
- Timeout: 30 giây
- Memory: 512 MB
- Handler: `get_articles_by_tag.lambda_handler`

![GetArticlesByTag Function](/images/5-Workshop/5.4-image-processing/GetArticlesByTag_Function.png)

**Luồng Logic:**

```
1. Parse và validate tag parameter
2. Scan GalleryPhotosTable
3. Filter photos có matching tag (trong memory)
4. Giới hạn kết quả
5. Trả về photos
```

**Ví dụ Response:**

```json
{
  "items": [
    {
      "photo_id": "article-123-image-1",
      "image_url": "articles/abc123/image.jpg",
      "tags": ["beach", "sunset", "ocean"],
      "status": "public",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

---

## Data Models

### GalleryPhotosTable

**Mục đích:** Lưu trữ photo metadata và tags cho gallery display

**Schema:**

| Field | Type | Mô tả |
|-------|------|-------|
| photo_id | String (PK) | ID duy nhất (vd: "article-123-image-1") |
| image_url | String | S3 key (vd: "articles/abc123/image.jpg") |
| tags | String[] | Kết hợp user tags + auto-detected tags |
| autoTags | String[] | Deprecated (backward compatibility) |
| status | String | "public" hoặc "private" |
| created_at | String | ISO 8601 timestamp |

**Indexes:**
- Primary Key: `photo_id`
- Không có GSIs

**Data Population:**
- Được populate bởi AI Service's `save_to_gallery` Lambda
- Trigger khi AI phát hiện labels trong ảnh
- Kết hợp user-provided tags với AI-detected tags

![Cấu trúc GalleryPhotosTable](/images/5-Workshop/5.4-image-processing/GalleryPhotosTable.png)

---

### GalleryTrendsTable

**Mục đích:** Theo dõi trending tags và thống kê độ phổ biến

**Schema:**

| Field | Type | Mô tả |
|-------|------|-------|
| tag_name | String (PK) | Tên tag (vd: "beach") |
| count | Number | Số lượng photos có tag này |
| cover_image | String | S3 key cho cover image (tùy chọn) |
| last_updated | String | ISO 8601 timestamp |

**Indexes:**
- Primary Key: `tag_name`
- Không có GSIs

**Data Population:**
- Được cập nhật bởi AI Service khi photos được thêm/xóa
- Tăng count khi photo mới với tag được thêm
- Giảm count khi photo bị xóa

![Cấu trúc GalleryTrendsTable](/images/5-Workshop/5.4-image-processing/GalleryTrendsTable.png)

---

## API Endpoints

### Endpoint 1: GET /gallery/trending

**Mục đích:** Lấy danh sách trending tags

**Request:**
```http
GET /gallery/trending?limit=20 HTTP/1.1
Host: {api-gateway-url}
```

**Query Parameters:**

| Parameter | Type | Bắt buộc | Mặc định | Mô tả |
|-----------|------|----------|----------|-------|
| limit | integer | Không | 20 | Số lượng tags tối đa trả về |

**Success Response (200):**
```json
{
  "items": [
    {
      "tag_name": "Beach",
      "count": 150,
      "cover_image": "articles/abc123/image.jpg",
      "last_updated": "2024-01-15T10:30:00Z"
    }
  ],
  "total_tags": 500
}
```

**Tích hợp Frontend:**

```typescript
async function getTrendingTags(limit: number = 20) {
  const response = await fetch(
    `${API_BASE_URL}/gallery/trending?limit=${limit}`
  );
  
  if (!response.ok) {
    throw new Error('Không thể lấy trending tags');
  }
  
  return await response.json();
}

// Sử dụng
const data = await getTrendingTags(10);
console.log(data.items);
```

### Hiển thị Gallery trên Frontend

**Giao diện Trending Tags:**

![Gallery Frontend Trending](/images/5-Workshop/5.4-image-processing/Gallery_Frontend_Trending.png)

**Giao diện Kết quả Tìm kiếm:**

![Gallery Frontend Search](/images/5-Workshop/5.4-image-processing/Gallery_Frontend_Search.png)

---

### Endpoint 2: GET /gallery/articles

**Mục đích:** Tìm kiếm photos theo tag

**Request:**
```http
GET /gallery/articles?tag=beach&limit=50 HTTP/1.1
Host: {api-gateway-url}
```

**Query Parameters:**

| Parameter | Type | Bắt buộc | Mặc định | Mô tả |
|-----------|------|----------|----------|-------|
| tag | string | Có | - | Tên tag để search (không phân biệt hoa thường) |
| limit | integer | Không | 50 | Số lượng photos tối đa trả về |

**Success Response (200):**
```json
{
  "items": [
    {
      "photo_id": "article-123-image-1",
      "image_url": "articles/abc123/image.jpg",
      "tags": ["beach", "sunset", "ocean"],
      "status": "public",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

---

## Phân tích Performance

### Metrics Performance Hiện tại

**Cấu hình Lambda:**

| Function | Timeout | Memory | Avg Execution | Cold Start |
|----------|---------|--------|---------------|------------|
| GetTrendingTags | 30s | 512 MB | ~2-5s | ~1-2s |
| GetArticlesByTag | 30s | 512 MB | ~3-8s | ~1-2s |

**DynamoDB Operations:**
- Cả hai functions đều dùng **Scan** operations (full table scan)
- Không sử dụng indexes
- Filtering và sorting trong memory

---

### Performance Bottlenecks

#### 1. Full Table Scans

**Vấn đề:** Scan toàn bộ table mỗi request

**Tác động:**
- Performance chậm với tables lớn (>10,000 items)
- Tiêu thụ DynamoDB read capacity cao
- Chi phí tăng

**Giải pháp:** Thêm Global Secondary Indexes (GSI)

#### 2. In-Memory Sorting (GetTrendingTags)

**Vấn đề:** Load tất cả tags vào memory, sort, rồi return top N

**Tác động:**
- Sử dụng memory không cần thiết
- Chậm với datasets lớn
- Rủi ro Lambda timeout

**Giải pháp:** Dùng DynamoDB GSI với count làm sort key

#### 3. In-Memory Filtering (GetArticlesByTag)

**Vấn đề:** Scan tất cả photos, filter theo tag trong Lambda

**Tác động:**
- Rất không hiệu quả
- Không scale được
- Latency cao

**Giải pháp:** Dùng DynamoDB GSI hoặc ElasticSearch

#### 4. Không có Caching

**Vấn đề:** Mỗi request đều query DynamoDB

**Tác động:**
- Load không cần thiết
- Chi phí cao hơn
- Response times chậm hơn

**Giải pháp:** Thêm caching layer (ElastiCache, CloudFront)

#### 5. Không có Pagination

**Vấn đề:** Trả về tất cả results (up to limit)

**Tác động:**
- Responses lớn
- Load times chậm
- UX kém

**Giải pháp:** Implement cursor-based pagination

---

## Khuyến nghị Tối ưu

### Cải thiện Ngắn hạn (Quick Wins)

1. **Thêm Caching**
   - Implement CloudFront caching cho API responses
   - TTL: 5 phút cho trending tags
   - Giảm DynamoDB load 90%+

2. **Tối ưu Lambda Memory**
   - Test với 256 MB thay vì 512 MB
   - Có thể giảm chi phí mà không ảnh hưởng performance

3. **Thêm Pagination**
   - Implement cursor-based pagination
   - Cải thiện frontend UX với infinite scroll

4. **Error Handling**
   - Thêm retry logic cho DynamoDB errors
   - Implement circuit breaker pattern

### Cải tiến Dài hạn

1. **Thêm DynamoDB GSIs**
   - CountIndex cho GalleryTrendsTable
   - TagIndex cho GalleryPhotosTable (cần denormalization)

2. **Implement ElasticSearch**
   - Tốt hơn cho complex tag queries
   - Full-text search capabilities
   - Faceted search support

3. **Thêm ElastiCache**
   - Redis cho frequently accessed data
   - Giảm DynamoDB costs
   - Cải thiện response times

4. **Implement Event-Driven Updates**
   - Dùng DynamoDB Streams
   - Tự động update cache khi data thay đổi
   - Duy trì cache consistency

---

## Ước tính Chi phí

### Chi phí Hiện tại (Ước tính)

**Giả định:**
- 10,000 photos trong gallery
- 500 unique tags
- 1,000 requests/ngày

**DynamoDB:**
- Read Capacity: ~$0.25/ngày (full scans)
- Storage: ~$0.01/ngày

**Lambda:**
- Invocations: ~$0.20/ngày
- Duration: ~$0.10/ngày

**API Gateway:**
- Requests: ~$0.04/ngày

**Tổng: ~$0.60/ngày hoặc ~$18/tháng**

### Chi phí Tối ưu (Với Caching)

**Với CloudFront + ElastiCache:**
- DynamoDB: ~$0.03/ngày (giảm 90%)
- Lambda: ~$0.03/ngày (giảm 90%)
- ElastiCache: ~$1.50/ngày (t3.micro)
- CloudFront: ~$0.02/ngày

**Tổng: ~$1.58/ngày hoặc ~$47/tháng**

**Lưu ý:** Chi phí upfront cao hơn nhưng performance và scalability tốt hơn.

---

## Điểm chính

1. **Gallery Service** cung cấp khám phá ảnh qua trending tags và search
2. **Implementation hiện tại** dùng full table scans (không hiệu quả)
3. **Performance bottlenecks** bao gồm không có caching, không có indexes, operations trong memory
4. **Quick wins** bao gồm thêm caching và pagination
5. **Dài hạn** nên thêm GSIs hoặc ElasticSearch để scalability tốt hơn
6. **Tối ưu chi phí** cần cân bằng giữa performance vs infrastructure costs

