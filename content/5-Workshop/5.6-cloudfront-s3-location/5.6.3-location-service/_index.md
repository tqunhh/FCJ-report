---
title: "AWS Location Service Integration"
date: 2025-12-09
weight: 3
chapter: false
pre: "<b>5.6.3. </b>"
---

## AWS Location Service (Map/Geocoding)

### Components Used in the Project

The project uses **Place Index** for geocoding, without requiring Map tiles.

**Place Index**: `TravelGuidePlaceIndex` (Esri)

**Main APIs:**
- `SearchPlaceIndexForPosition` (reverse geocoding)
- `SearchPlaceIndexForText` (forward geocoding)

---

## Place Index Configuration

### Creating Place Index

**AWS Console Steps:**
1. Navigate to Amazon Location Service
2. Create Place Index
3. Configure:
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

## Backend Integration

### How Backend Calls Location Service

In `functions/utils/location_service.py`:

**Logic flow:**
1. Check DynamoDB cache
2. If cache miss → call AWS Location
3. If error/no results → fallback to Nominatim
4. Save to cache with TTL (default 30 days)

### Lambda Functions Using Location Service

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

### Python Code Example

```python
import boto3
import json
from datetime import datetime, timedelta

location_client = boto3.client('location')
dynamodb = boto3.resource('dynamodb')
cache_table = dynamodb.Table(os.environ['LOCATION_CACHE_TABLE'])
PLACE_INDEX = os.environ['PLACE_INDEX_NAME']

def reverse_geocode(latitude, longitude):
    """Convert coordinates to place name"""
    
    # Generate cache key
    cache_key = f"reverse:{latitude:.6f},{longitude:.6f}"
    
    # Check cache
    try:
        response = cache_table.get_item(Key={'cacheKey': cache_key})
        if 'Item' in response:
            cached_data = response['Item']
            # Check if not expired
            if datetime.now().timestamp() < cached_data['expiresAt']:
                print(f"Cache HIT: {cache_key}")
                return json.loads(cached_data['data'])
    except Exception as e:
        print(f"Cache read error: {e}")
    
    print(f"Cache MISS: {cache_key}")
    
    # Call AWS Location Service
    try:
        response = location_client.search_place_index_for_position(
            IndexName=PLACE_INDEX,
            Position=[longitude, latitude],  # Note: [lng, lat]
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
            
            # Save to cache
            save_to_cache(cache_key, result)
            return result
            
    except Exception as e:
        print(f"AWS Location error: {e}")
        # Fallback to Nominatim
        return fallback_nominatim(latitude, longitude)
    
    return None

def forward_geocode(text):
    """Convert place name to coordinates"""
    
    cache_key = f"forward:{text}"
    
    # Check cache (similar to reverse)
    # ...
    
    # Call AWS Location Service
    try:
        response = location_client.search_place_index_for_text(
            IndexName=PLACE_INDEX,
            Text=text,
            MaxResults=5
        )
        
        results = []
        for item in response['Results']:
            place = item['Place']
            results.append({
                'label': place.get('Label', ''),
                'position': place['Geometry']['Point'],  # [lng, lat]
                'country': place.get('Country', '')
            })
        
        # Save to cache
        save_to_cache(cache_key, results)
        return results
        
    except Exception as e:
        print(f"AWS Location error: {e}")
        return []

def save_to_cache(key, data, ttl_days=30):
    """Save geocoding result to DynamoDB cache"""
    try:
        expires_at = int((datetime.now() + timedelta(days=ttl_days)).timestamp())
        cache_table.put_item(
            Item={
                'cacheKey': key,
                'data': json.dumps(data),
                'expiresAt': expires_at,
                'createdAt': int(datetime.now().timestamp())
            }
        )
        print(f"Saved to cache: {key}")
    except Exception as e:
        print(f"Cache write error: {e}")

def fallback_nominatim(latitude, longitude):
    """Fallback to Nominatim if AWS Location fails"""
    import requests
    
    try:
        url = f"https://nominatim.openstreetmap.org/reverse"
        params = {
            'lat': latitude,
            'lon': longitude,
            'format': 'json'
        }
        headers = {'User-Agent': 'TravelGuideApp/1.0'}
        
        response = requests.get(url, params=params, headers=headers, timeout=5)
        if response.status_code == 200:
            data = response.json()
            return {
                'label': data.get('display_name', ''),
                'municipality': data.get('address', {}).get('city', ''),
                'country': data.get('address', {}).get('country', ''),
                'region': data.get('address', {}).get('state', '')
            }
    except Exception as e:
        print(f"Nominatim fallback error: {e}")
    
    return None
```

---

## DynamoDB Cache for Cost Optimization

### LocationCacheTable Structure

![Location Cache Table](/images/5-Workshop/5.6-cloudfront-s3-location/LocationCacheTable.png)

**Table Schema:**

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
- `cacheKey` (String, Primary Key): `reverse:lat,lng` or `forward:text`
- `data` (String): JSON-encoded geocoding result
- `expiresAt` (Number): Unix timestamp for TTL
- `createdAt` (Number): Creation timestamp

**Benefits:**
- Reduces AWS Location Service requests
- Saves costs (Location Service is request-based)
- Faster response times
- Automatic cleanup via TTL

---

## IAM Permissions

### Lambda Execution Role

Minimum IAM policy for Lambda:

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

**SAM Policy Templates:**

```yaml
Policies:
  - DynamoDBCrudPolicy:
      TableName: !Ref LocationCacheTable
  - Statement:
      - Effect: Allow
        Action:
          - geo:SearchPlaceIndexForPosition
          - geo:SearchPlaceIndexForText
        Resource: !GetAtt TravelGuidePlaceIndex.Arn
```

---

## Operations & Cost Optimization

### Monitoring Cache Performance

**Metrics to track:**
- Cache hit ratio
- Cache miss ratio
- AWS Location Service request count
- Average response time

**CloudWatch Custom Metrics:**

```python
import boto3

cloudwatch = boto3.client('cloudwatch')

def log_cache_metric(metric_name, value):
    cloudwatch.put_metric_data(
        Namespace='TravelGuide/Location',
        MetricData=[
            {
                'MetricName': metric_name,
                'Value': value,
                'Unit': 'Count'
            }
        ]
    )

# Usage
log_cache_metric('CacheHit', 1)
log_cache_metric('CacheMiss', 1)
log_cache_metric('LocationServiceCall', 1)
```

### Cost Optimization Strategies

1. **Increase Cache TTL**
   - For stable locations: 90 days
   - For changing data: 7-30 days

2. **Cache Popular Queries**
   - Pre-populate cache for common locations
   - Reduce cold start costs

3. **Batch Requests** (if applicable)
   - Group multiple geocoding requests
   - Reduce API calls

4. **Monitor Usage**
   - Set CloudWatch alarms for high request counts
   - Review monthly Location Service costs

5. **Fallback Strategy**
   - Use Nominatim for non-critical requests
   - Reserve AWS Location for premium features

---

## Frontend Integration (Optional)

### Using AWS Location Maps

If you want to use AWS Location Maps (instead of OSM/Esri public tiles), you have two approaches:

#### 1. API Key (Amazon Location API Key)
- Easy integration
- Suitable for public web
- Has quota/limits

#### 2. Cognito + SigV4 Signed Requests
- More secure
- More complex setup
- Better for authenticated users

### Leaflet Integration Example

```javascript
import L from 'leaflet';
import { Signer } from '@aws-amplify/core';

// With API Key
const map = L.map('map').setView([10.8231, 106.6297], 13);

L.tileLayer(
  'https://maps.geo.ap-southeast-1.amazonaws.com/maps/v0/maps/{mapName}/tiles/{z}/{x}/{y}?key={apiKey}',
  {
    attribution: '© Amazon Location Service',
    mapName: 'TravelGuideMap',
    apiKey: 'YOUR_API_KEY'
  }
).addTo(map);

// With Cognito (SigV4)
// More complex - requires signing each tile request
// See AWS documentation for full implementation
```

---

## Troubleshooting

### Common Issues

#### 1. No Results from Location Service

**Causes:**
- Invalid coordinates
- Location not in Esri database
- Incorrect Place Index configuration

**Solution:**
- Validate lat/lng ranges (-90 to 90, -180 to 180)
- Use fallback to Nominatim
- Check Place Index data source

#### 2. High Costs

**Causes:**
- Low cache hit ratio
- Too many unique queries
- Short cache TTL

**Solution:**
- Increase cache TTL
- Pre-populate common locations
- Review query patterns

#### 3. Slow Response Times

**Causes:**
- Cold Lambda start
- Cache miss
- Network latency

**Solution:**
- Use provisioned concurrency for Lambda
- Pre-warm cache
- Implement async geocoding for non-critical paths

#### 4. Cache Not Working

**Causes:**
- DynamoDB permissions missing
- TTL not configured
- Cache key mismatch

**Solution:**
- Verify IAM permissions
- Check DynamoDB TTL settings
- Debug cache key generation

---

## Best Practices

1. **Always Use Cache**
   - Reduces costs significantly
   - Improves performance

2. **Implement Fallback**
   - Nominatim for reliability
   - Handle service outages gracefully

3. **Validate Input**
   - Check coordinate ranges
   - Sanitize text queries

4. **Monitor Usage**
   - Track request counts
   - Set cost alerts

5. **Optimize TTL**
   - Longer for stable data
   - Shorter for dynamic data

6. **Log Cache Performance**
   - Hit/miss ratios
   - Identify optimization opportunities

---

## Key Takeaways

1. **AWS Location Service** provides geocoding without managing infrastructure
2. **DynamoDB cache** dramatically reduces costs
3. **Fallback strategy** ensures reliability
4. **Proper IAM permissions** are essential
5. **Monitoring** helps optimize performance and costs
6. **Cache TTL** balances freshness and cost

