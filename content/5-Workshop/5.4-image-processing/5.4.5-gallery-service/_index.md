---
title: "Gallery Service - Photo Discovery"
date: 2025-12-09
weight: 5
chapter: false
pre: "<b>5.4.5. </b>"
---

## Gallery Service Overview

Gallery Service provides photo discovery functionality through trending tags and tag-based search. It consumes data from AI Service's label detection and presents it through user-friendly APIs.

### Purpose and Use Cases

**Main Functions:**
1. **Trending Tags**: Display most popular tags based on photo count
2. **Photo Gallery**: Search photos by specific tags

**Position in System:**
- Consumes data from AI Service (auto-detected tags)
- Provides API for Frontend to display gallery
- Reads from DynamoDB tables populated by AI Service

---

## Architecture

### Components Overview

![Gallery API Gateway](/images/5-Workshop/5.4-image-processing/Gallery_API_Gateway.png)

The Gallery Service consists of:
- **2 Lambda Functions**: GetTrendingTags, GetArticlesByTag
- **1 API Gateway**: REST API with 2 endpoints
- **1 Lambda Layer**: Shared dependencies (boto3, CORS utilities)
- **2 DynamoDB Tables**: GalleryPhotosTable, GalleryTrendsTable (from Core Stack)

### Data Flow

```
User Uploads Image
    ↓
Article Service → S3 Bucket
    ↓
S3 Event → AI Service
    ↓
Label Detection
    ↓
├─→ Save to GalleryPhotosTable
└─→ Update GalleryTrendsTable (increment count)
    ↓
Gallery Service (Read-only)
    ↓
Frontend Display
```

### Dependencies with Core Stack

Gallery Service imports from Core Stack:

```yaml
# From core-infra stack
Imports:
  - GalleryPhotosTableName
  - GalleryTrendsTableName
```

**Deployment Order:**
1. Core Stack (creates tables)
2. AI Service (populates tables)
3. Gallery Service (reads tables)

---

## Lambda Functions

### Function 1: GetTrendingTagsFunction

**Purpose:** Return list of trending tags sorted by popularity

**Configuration:**
- Runtime: Python 3.11
- Timeout: 30 seconds
- Memory: 512 MB
- Handler: `get_trending_tags.lambda_handler`

![GetTrendingTags Function](/images/5-Workshop/5.4-image-processing/GetTrendingTags_Function.png)

**Logic Flow:**

```
1. Parse query parameters (limit)
2. Scan all tags from GalleryTrendsTable
3. Sort by count (descending) in-memory
4. Return top N tags
```

**Code Structure:**

```python
def lambda_handler(event, context):
    # Parse parameters
    limit = int(params.get("limit", 20))
    
    # Scan all tags
    all_tags = []
    while True:
        response = table.scan(**scan_kwargs)
        all_tags.extend(response.get('Items', []))
        if 'LastEvaluatedKey' not in response:
            break
    
    # Sort by count
    all_tags.sort(key=lambda x: x.get('count', 0), reverse=True)
    
    # Return top N
    return {
        'statusCode': 200,
        'body': json.dumps({
            'items': all_tags[:limit],
            'total_tags': len(all_tags)
        })
    }
```

**Response Example:**

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

**Purpose:** Search photos with specific tag

**Configuration:**
- Runtime: Python 3.11
- Timeout: 30 seconds
- Memory: 512 MB
- Handler: `get_articles_by_tag.lambda_handler`

![GetArticlesByTag Function](/images/5-Workshop/5.4-image-processing/GetArticlesByTag_Function.png)

**Logic Flow:**

```
1. Parse and validate tag parameter
2. Scan GalleryPhotosTable
3. Filter photos with matching tag (in-memory)
4. Limit results
5. Return photos
```

**Code Structure:**

```python
def lambda_handler(event, context):
    # Validate parameters
    tag = params.get("tag", "").strip().lower()
    limit = int(params.get("limit", 50))
    
    if not tag:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': 'tag parameter is required'})
        }
    
    # Scan and filter
    matching_photos = []
    while len(matching_photos) < limit:
        response = photos_table.scan(**scan_kwargs)
        
        for item in response.get('Items', []):
            photo_tags = [t.lower() for t in item.get('tags', [])]
            if tag in photo_tags:
                matching_photos.append(item)
        
        if 'LastEvaluatedKey' not in response:
            break
    
    return {
        'statusCode': 200,
        'body': json.dumps({'items': matching_photos[:limit]})
    }
```

**Response Example:**

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

**Purpose:** Store photo metadata and tags for gallery display

**Schema:**

| Field | Type | Description |
|-------|------|-------------|
| photo_id | String (PK) | Unique identifier (e.g., "article-123-image-1") |
| image_url | String | S3 key (e.g., "articles/abc123/image.jpg") |
| tags | String[] | Combined user tags + auto-detected tags |
| autoTags | String[] | Deprecated (backward compatibility) |
| status | String | "public" or "private" |
| created_at | String | ISO 8601 timestamp |
| createdAt | String | Alias for frontend |

**Indexes:**
- Primary Key: `photo_id`
- No GSIs (Global Secondary Indexes)

**Data Population:**
- Populated by AI Service's `save_to_gallery` Lambda
- Triggered when AI detects labels in images
- Combines user-provided tags with AI-detected tags

![GalleryPhotosTable Structure](/images/5-Workshop/5.4-image-processing/GalleryPhotosTable.png)

---

### GalleryTrendsTable

**Purpose:** Track trending tags and popularity statistics

**Schema:**

| Field | Type | Description |
|-------|------|-------------|
| tag_name | String (PK) | Tag name (e.g., "beach") |
| count | Number | Number of photos with this tag |
| cover_image | String | S3 key for cover image (optional) |
| last_updated | String | ISO 8601 timestamp |

**Indexes:**
- Primary Key: `tag_name`
- No GSIs

**Data Population:**
- Updated by AI Service when photos are added/removed
- Increment count when new photo with tag is added
- Decrement count when photo is deleted

![GalleryTrendsTable Structure](/images/5-Workshop/5.4-image-processing/GalleryTrendsTable.png)

---

## API Endpoints

### Endpoint 1: GET /gallery/trending

**Purpose:** Get list of trending tags

**Request:**
```http
GET /gallery/trending?limit=20 HTTP/1.1
Host: {api-gateway-url}
```

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| limit | integer | No | 20 | Maximum number of tags to return |

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

**Error Response (500):**
```json
{
  "error": "Internal error: Gallery Trends table not configured"
}
```

**Frontend Integration:**

```typescript
async function getTrendingTags(limit: number = 20) {
  const response = await fetch(
    `${API_BASE_URL}/gallery/trending?limit=${limit}`
  );
  
  if (!response.ok) {
    throw new Error('Failed to fetch trending tags');
  }
  
  return await response.json();
}

// Usage
const data = await getTrendingTags(10);
console.log(data.items);
```

---

### Endpoint 2: GET /gallery/articles

**Purpose:** Search photos by tag

**Request:**
```http
GET /gallery/articles?tag=beach&limit=50 HTTP/1.1
Host: {api-gateway-url}
```

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| tag | string | Yes | - | Tag name to search (case-insensitive) |
| limit | integer | No | 50 | Maximum number of photos to return |

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

**Error Responses:**

```json
// 400 - Bad Request
{
  "error": "tag parameter is required"
}

// 500 - Internal Server Error
{
  "error": "Internal error: Gallery Photos table not configured"
}
```

**Frontend Integration:**

```typescript
async function getPhotosByTag(tag: string, limit: number = 50) {
  const response = await fetch(
    `${API_BASE_URL}/gallery/articles?tag=${encodeURIComponent(tag)}&limit=${limit}`
  );
  
  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'Failed to fetch photos');
  }
  
  return await response.json();
}

// Usage
const photos = await getPhotosByTag('beach', 20);
console.log(photos.items);
```

### Frontend Gallery Display

**Trending Tags View:**

![Gallery Frontend Trending](/images/5-Workshop/5.4-image-processing/Gallery_Frontend_Trending.png)

**Search Results View:**

![Gallery Frontend Search](/images/5-Workshop/5.4-image-processing/Gallery_Frontend_Search.png)

---

## Performance Analysis

### Current Performance Metrics

**Lambda Configuration:**

| Function | Timeout | Memory | Avg Execution | Cold Start |
|----------|---------|--------|---------------|------------|
| GetTrendingTags | 30s | 512 MB | ~2-5s | ~1-2s |
| GetArticlesByTag | 30s | 512 MB | ~3-8s | ~1-2s |

**DynamoDB Operations:**
- Both functions use **Scan** operations (full table scan)
- No indexes utilized
- In-memory filtering and sorting

---

### Performance Bottlenecks

#### 1. Full Table Scans

**Issue:** Scan entire table on every request

**Impact:**
- Slow performance with large tables (>10,000 items)
- High DynamoDB read capacity consumption
- Increased costs

**Solution:** Add Global Secondary Indexes (GSI)

#### 2. In-Memory Sorting (GetTrendingTags)

**Issue:** Load all tags into memory, sort, then return top N

**Impact:**
- Unnecessary memory usage
- Slow with large datasets
- Lambda timeout risk

**Solution:** Use DynamoDB GSI with count as sort key

```yaml
# Recommended GSI
GalleryTrendsTable:
  GlobalSecondaryIndexes:
    - IndexName: CountIndex
      KeySchema:
        - AttributeName: dummy_pk  # All items have same value
          KeyType: HASH
        - AttributeName: count
          KeyType: RANGE
      Projection:
        ProjectionType: ALL
```

#### 3. In-Memory Filtering (GetArticlesByTag)

**Issue:** Scan all photos, filter by tag in Lambda

**Impact:**
- Very inefficient
- Doesn't scale
- High latency

**Solution:** Use DynamoDB GSI or ElasticSearch

```yaml
# Recommended GSI
GalleryPhotosTable:
  GlobalSecondaryIndexes:
    - IndexName: TagIndex
      KeySchema:
        - AttributeName: tag
          KeyType: HASH
        - AttributeName: created_at
          KeyType: RANGE
      Projection:
        ProjectionType: ALL
```

**Note:** DynamoDB doesn't support array attributes in GSI. Need to denormalize data or use ElasticSearch.

#### 4. No Caching

**Issue:** Every request queries DynamoDB

**Impact:**
- Unnecessary load
- Higher costs
- Slower response times

**Solution:** Add caching layer

**Options:**
- **ElastiCache Redis**: For frequently accessed data
- **CloudFront**: For API responses
- **Lambda in-memory cache**: For small datasets

**Example with ElastiCache:**

```python
import redis

redis_client = redis.Redis(host=REDIS_HOST, port=6379)
CACHE_TTL = 300  # 5 minutes

def get_trending_tags_cached(limit):
    cache_key = f"trending_tags:{limit}"
    
    # Try cache first
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Query DynamoDB
    result = get_trending_tags_from_db(limit)
    
    # Save to cache
    redis_client.setex(cache_key, CACHE_TTL, json.dumps(result))
    
    return result
```

#### 5. No Pagination

**Issue:** Return all results (up to limit)

**Impact:**
- Large responses
- Slow load times
- Poor user experience

**Solution:** Implement cursor-based pagination

```python
def lambda_handler(event, context):
    params = event.get('queryStringParameters', {})
    limit = int(params.get('limit', 20))
    cursor = params.get('cursor')  # LastEvaluatedKey from previous request
    
    scan_kwargs = {'Limit': limit}
    if cursor:
        scan_kwargs['ExclusiveStartKey'] = json.loads(base64.b64decode(cursor))
    
    response = table.scan(**scan_kwargs)
    
    result = {
        'items': response['Items'],
        'cursor': None
    }
    
    if 'LastEvaluatedKey' in response:
        result['cursor'] = base64.b64encode(
            json.dumps(response['LastEvaluatedKey']).encode()
        ).decode()
    
    return {'statusCode': 200, 'body': json.dumps(result)}
```

---

## Optimization Recommendations

### Short-term Improvements (Quick Wins)

1. **Add Caching**
   - Implement CloudFront caching for API responses
   - TTL: 5 minutes for trending tags
   - Reduces DynamoDB load by 90%+

2. **Optimize Lambda Memory**
   - Test with 256 MB instead of 512 MB
   - May reduce costs without impacting performance

3. **Add Pagination**
   - Implement cursor-based pagination
   - Improve frontend UX with infinite scroll

4. **Error Handling**
   - Add retry logic for DynamoDB errors
   - Implement circuit breaker pattern

### Long-term Enhancements

1. **Add DynamoDB GSIs**
   - CountIndex for GalleryTrendsTable
   - TagIndex for GalleryPhotosTable (requires denormalization)

2. **Implement ElasticSearch**
   - Better for complex tag queries
   - Full-text search capabilities
   - Faceted search support

3. **Add ElastiCache**
   - Redis for frequently accessed data
   - Reduce DynamoDB costs
   - Improve response times

4. **Implement Event-Driven Updates**
   - Use DynamoDB Streams
   - Update cache automatically when data changes
   - Maintain cache consistency

---

## Cost Estimation

### Current Costs (Estimated)

**Assumptions:**
- 10,000 photos in gallery
- 500 unique tags
- 1,000 requests/day

**DynamoDB:**
- Read Capacity: ~$0.25/day (full scans)
- Storage: ~$0.01/day

**Lambda:**
- Invocations: ~$0.20/day
- Duration: ~$0.10/day

**API Gateway:**
- Requests: ~$0.04/day

**Total: ~$0.60/day or ~$18/month**

### Optimized Costs (With Caching)

**With CloudFront + ElastiCache:**
- DynamoDB: ~$0.03/day (90% reduction)
- Lambda: ~$0.03/day (90% reduction)
- ElastiCache: ~$1.50/day (t3.micro)
- CloudFront: ~$0.02/day

**Total: ~$1.58/day or ~$47/month**

**Note:** Higher upfront cost but better performance and scalability.

---

## Key Takeaways

1. **Gallery Service** provides photo discovery through trending tags and search
2. **Current implementation** uses full table scans (inefficient)
3. **Performance bottlenecks** include no caching, no indexes, in-memory operations
4. **Quick wins** include adding caching and pagination
5. **Long-term** should add GSIs or ElasticSearch for better scalability
6. **Cost optimization** requires balancing performance vs infrastructure costs

