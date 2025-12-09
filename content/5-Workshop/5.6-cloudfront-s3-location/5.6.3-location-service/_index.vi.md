---
title: "Tích hợp AWS Location Service"
date: 2025-12-09
weight: 3
chapter: false
pre: "<b>5.6.3. </b>"
---

## AWS Location Service (Map/Geocoding)

### Thành phần Sử dụng trong Dự án

Dự án sử dụng **Place Index** cho geocoding, không bắt buộc phải dùng Map tiles.

**Place Index**: `TravelGuidePlaceIndex` (Esri)

**APIs chính:**
- `SearchPlaceIndexForPosition` (reverse geocoding)
- `SearchPlaceIndexForText` (forward geocoding)

---

## Cấu hình Place Index

### Tạo Place Index

**Các bước trên AWS Console:**
1. Truy cập Amazon Location Service
2. Tạo Place Index
3. Cấu hình:
   - Name: `TravelGuidePlaceIndex`
   - Data provider: **Esri**
   - Pricing plan: **RequestBasedUsage**
   - Intended use: `SingleUse`

**SAM Template:**

```yaml
TravelGuidePlaceIndex:
  Type: AWS::Location::PlaceIndex
  Properties:
    IndexName: TravelGuidePlaceIndex
    DataSource: Esri
    PricingPlan: RequestBasedUsage
    DataSourceConfiguration:
      IntendedUse: SingleUse
```

---

## Tích hợp Backend

### Cách Backend gọi Location Service

Trong `functions/utils/location_service.py`:

**Luồng logic:**
1. Kiểm tra DynamoDB cache
2. Nếu cache miss → gọi AWS Location
3. Nếu lỗi/không có kết quả → fallback Nominatim
4. Lưu vào cache với TTL (mặc định 30 ngày)

### Lambda Functions sử dụng Location Service

**Functions:**
- `CreateArticleFunction`
- `UpdateArticleFunction`

**Environment Variables:**
```yaml
Environment:
  Variables:
    PLACE_INDEX_NAME: !Ref TravelGuidePlaceIndex
    LOCATION_CACHE_TABLE: !Ref LocationCacheTable
    USE_AWS_LOCATION: 'true'
```

### Ví dụ Python Code

```python
import boto3
import json
from datetime import datetime, timedelta

location_client = boto3.client('location')
dynamodb = boto3.resource('dynamodb')
cache_table = dynamodb.Table(os.environ['LOCATION_CACHE_TABLE'])
PLACE_INDEX = os.environ['PLACE_INDEX_NAME']

def reverse_geocode(latitude, longitude):
    """Chuyển đổi tọa độ thành tên địa điểm"""
    
    # Tạo cache key
    cache_key = f"reverse:{latitude:.6f},{longitude:.6f}"
    
    # Kiểm tra cache
    try:
        response = cache_table.get_item(Key={'cacheKey': cache_key})
        if 'Item' in response:
            cached_data = response['Item']
            if datetime.now().timestamp() < cached_data['expiresAt']:
                print(f"Cache HIT: {cache_key}")
                return json.loads(cached_data['data'])
    except Exception as e:
        print(f"Lỗi đọc cache: {e}")
    
    print(f"Cache MISS: {cache_key}")
    
    # Gọi AWS Location Service
    try:
        response = location_client.search_place_index_for_position(
            IndexName=PLACE_INDEX,
            Position=[longitude, latitude],  # Lưu ý: [lng, lat]
            MaxResults=1
        )
        
        if response['Results']:
            place = response['Results'][0]['Place']
            result = {
                'label': place.get('Label', ''),
                'municipality': place.get('Municipality', ''),
                'country': place.get('Country', ''),
                'region': place.get('Region', '')
            }
            
            # Lưu vào cache
            save_to_cache(cache_key, result)
            return result
            
    except Exception as e:
        print(f"Lỗi AWS Location: {e}")
        return fallback_nominatim(latitude, longitude)
    
    return None
```

---

## DynamoDB Cache để Tối ưu Chi phí

### Cấu trúc LocationCacheTable

![Location Cache Table](/images/5-Workshop/5.6-cloudfront-s3-location/LocationCacheTable.png)

**Schema Table:**

```yaml
LocationCacheTable:
  Type: AWS::DynamoDB::Table
  Properties:
    TableName: LocationCache
    BillingMode: PAY_PER_REQUEST
    AttributeDefinitions:
      - AttributeName: cacheKey
        AttributeType: S
    KeySchema:
      - AttributeName: cacheKey
        KeyType: HASH
    TimeToLiveSpecification:
      Enabled: true
      AttributeName: expiresAt
```

**Attributes:**
- `cacheKey` (String, Primary Key): `reverse:lat,lng` hoặc `forward:text`
- `data` (String): Kết quả geocoding dạng JSON
- `expiresAt` (Number): Unix timestamp cho TTL
- `createdAt` (Number): Timestamp tạo

**Lợi ích:**
- Giảm requests đến AWS Location Service
- Tiết kiệm chi phí (Location Service tính theo request)
- Thời gian response nhanh hơn
- Tự động cleanup qua TTL

---

## Quyền IAM

### Lambda Execution Role

IAM policy tối thiểu cho Lambda:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "geo:SearchPlaceIndexForPosition",
        "geo:SearchPlaceIndexForText"
      ],
      "Resource": "arn:aws:geo:ap-southeast-1:*:place-index/TravelGuidePlaceIndex"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:*:table/LocationCache"
    }
  ]
}
```

---

## Vận hành & Tối ưu Chi phí

### Giám sát Hiệu suất Cache

**Metrics cần theo dõi:**
- Tỷ lệ cache hit
- Tỷ lệ cache miss
- Số lượng AWS Location Service requests
- Thời gian response trung bình

### Chiến lược Tối ưu Chi phí

1. **Tăng Cache TTL**
   - Cho locations ổn định: 90 ngày
   - Cho dữ liệu thay đổi: 7-30 ngày

2. **Cache Popular Queries**
   - Pre-populate cache cho locations phổ biến
   - Giảm chi phí cold start

3. **Giám sát Usage**
   - Đặt CloudWatch alarms cho request counts cao
   - Review chi phí Location Service hàng tháng

4. **Fallback Strategy**
   - Dùng Nominatim cho requests không quan trọng
   - Dành AWS Location cho premium features

---

## Xử lý Sự cố

### Các vấn đề thường gặp

#### 1. Không có Kết quả từ Location Service

**Nguyên nhân:**
- Tọa độ không hợp lệ
- Location không có trong database Esri
- Cấu hình Place Index sai

**Giải pháp:**
- Validate phạm vi lat/lng (-90 đến 90, -180 đến 180)
- Dùng fallback Nominatim
- Kiểm tra data source của Place Index

#### 2. Chi phí Cao

**Nguyên nhân:**
- Tỷ lệ cache hit thấp
- Quá nhiều queries độc nhất
- Cache TTL ngắn

**Giải pháp:**
- Tăng cache TTL
- Pre-populate locations phổ biến
- Review query patterns

#### 3. Response Chậm

**Nguyên nhân:**
- Lambda cold start
- Cache miss
- Network latency

**Giải pháp:**
- Dùng provisioned concurrency cho Lambda
- Pre-warm cache
- Implement async geocoding cho non-critical paths

---

## Best Practices

1. **Luôn Dùng Cache** - Giảm chi phí đáng kể, cải thiện performance
2. **Implement Fallback** - Nominatim cho reliability
3. **Validate Input** - Kiểm tra phạm vi coordinates
4. **Monitor Usage** - Theo dõi request counts, đặt cost alerts
5. **Optimize TTL** - Dài hơn cho stable data, ngắn hơn cho dynamic data
6. **Log Cache Performance** - Hit/miss ratios để tối ưu

---

## Điểm chính

1. **AWS Location Service** cung cấp geocoding không cần quản lý infrastructure
2. **DynamoDB cache** giảm chi phí đáng kể
3. **Fallback strategy** đảm bảo reliability
4. **IAM permissions phù hợp** là thiết yếu
5. **Monitoring** giúp tối ưu performance và costs
6. **Cache TTL** cân bằng giữa freshness và cost

