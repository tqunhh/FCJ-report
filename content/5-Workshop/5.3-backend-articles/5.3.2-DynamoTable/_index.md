---
title: " DynamoDB Tables Overview"
date: 2025-12-08
chapter: false
weight: 2
pre : " <b>5.3.2.  </b> "
---

# 5.4 Overview of DynamoDB Tables

This section explains all DynamoDB tables defined inside the backend architecture.  
Each table serves a specific role in powering article management, user interactions, gallery rendering, location services, and user profiles.

#### Complete DynamoDB Tables Architecture

![Full DynamoDB Tables](/images/5-Workshop/5.3-backend-articles/Full_Dynamtable.png)

---

## **1. ArticlesTable**

![ArticlesTable Structure](/images/5-Workshop/5.3-backend-articles/ArticlesTable.png)
**Purpose:**  
Stores all articles created by users. Each record represents a complete travel post including metadata, visibility settings, timestamps, and ownership.

**Primary Key:**  
- `articleId` (HASH)

**Attributes:**  
- `visibility` — public/private  
- `createdAt` — ISO timestamp  
- `ownerId` — ID of the user who created the article  
- `status` — draft/published/flagged  

**Indexes:**  
- **gsi_visibility_createdAt (visibility, createdAt)** → For listing public articles in the home feed.  
- **gsi_owner_createdAt (ownerId, createdAt)** → For user profile pages.  
- **gsi_status_createdAt (status, createdAt)** → For admin filtering and moderation.

---

## **2. UserFavoritesTable**

![UserFavoritesTable Structure](/images/5-Workshop/5.3-backend-articles/UserFavoritesTable.png)

**Purpose:**  
Stores the relationship between a user and the articles they have favorited.

**Primary Key (Composite):**  
- `userId` (HASH)  
- `articleId` (RANGE)

This structure allows efficient:  
- Checking whether a user has favorited an article  
- Listing all favorites of the current user  

---

## **3. GalleryPhotosTable**

![GalleryPhotosTable Structure](/images/5-Workshop/5.3-backend-articles/GalleryPhotosTable.png)

**Purpose:**  
Stores metadata for individual images extracted from articles.  
Used for gallery browsing, filtering by tags, and building trending metrics.

**Primary Key:**  
- `photo_id` (HASH)

**Metadata may include:**  
- `articleId` — which article the photo belongs to  
- S3 image key  
- detected tags  
- geolocation or timestamps

---

## **4. GalleryTrendsTable**

![GalleryTrendsTable Structure](/images/5-Workshop/5.3-backend-articles/GalleryTrendsTable.png)

**Purpose:**  
Maintains tag popularity statistics for the gallery.  
The article/image ingestion process updates counts within this table.

**Primary Key:**  
- `tag_name` (HASH)

**Usage:**  
- Provides trending tags for the homepage  
- Enables tag-based article recommendation  

---

## **5. UserProfilesTable**

![UserProfilesTable Structure](/images/5-Workshop/5.3-backend-articles/UserProfilesTable.png)

**Purpose:**  
Stores public profile information of users displayed in article feeds and profile pages.

**Primary Key:**  
- `userId` (HASH)

**Attributes include:**  
- avatar image URL  
- full name  
- bio/description  

This table keeps profile data separate from Cognito, allowing customization and public display without exposing private identity data.

---

## **6. LocationCacheTable**

![LocationCacheTable Structure](/images/5-Workshop/5.3-backend-articles/LocationCacheTable.png)

**Purpose:**  
Caches results from AWS Location Service's reverse-geocoding to reduce cost and improve performance.

**Primary Key:**  
- `geohash` (HASH)

**Attributes:**  
- human-readable location  
- `TTL` — automatically deletes outdated entries

---

# **Summary**

Together, these DynamoDB tables form the storage foundation for:  
- Article CRUD  
- Favorites  
- Gallery features  
- User profile rendering  
- Location lookup optimization  

They are optimized for serverless workloads with high scalability and low latency.

