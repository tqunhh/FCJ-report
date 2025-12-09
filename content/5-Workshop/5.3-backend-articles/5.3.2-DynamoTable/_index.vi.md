---
title: " bảng DynamoDB"
date: 2025-12-08
weight: 2
chapter: false
pre : " <b>5.3.2.  </b> "
---

# 5.4 Giải thích các bảng DynamoDB

Phần này mô tả toàn bộ các bảng DynamoDB được định nghĩa trong kiến trúc backend.  
Mỗi bảng phục vụ một vai trò cụ thể: quản lý bài viết, hiển thị gallery, xử lý yêu thích, cache vị trí và lưu thông tin hồ sơ người dùng.

#### Kiến trúc Toàn bộ Bảng DynamoDB

![Toàn bộ Bảng DynamoDB](/images/5-Workshop/5.3-backend-articles/Full_Dynamtable.png)

---

## 1. ArticlesTable

![Cấu trúc ArticlesTable](/images/5-Workshop/5.3-backend-articles/ArticlesTable.png)

**Mục đích:**  
Lưu toàn bộ bài viết do người dùng tạo. Mỗi bản ghi đại diện cho một bài review đầy đủ: metadata, quyền hiển thị, thời gian tạo, trạng thái và người sở hữu.

**Primary Key:**  
- `articleId` (HASH)

**Thuộc tính:**  
- `visibility` — public/private  
- `createdAt` — thời gian tạo  
- `ownerId` — ID người tạo bài viết  
- `status` — draft/published/flagged  

**Các GSI:**  
- **gsi_visibility_createdAt** → hiển thị bài public trên feed trang chủ  
- **gsi_owner_createdAt** → lấy bài của một user trong trang profile  
- **gsi_status_createdAt** → phục vụ lọc theo trạng thái (quản trị)  

---

## 2. UserFavoritesTable

![Cấu trúc UserFavoritesTable](/images/5-Workshop/5.3-backend-articles/UserFavoritesTable.png)

**Mục đích:**  
Lưu mối quan hệ yêu thích giữa người dùng và bài viết.

**Primary Key (kép):**  
- `userId` (HASH)  
- `articleId` (RANGE)

Bảng này giúp:  
- Kiểm tra user đã thích bài chưa  
- Lấy danh sách tất cả bài đã thích  

---

## 3. GalleryPhotosTable

![Cấu trúc GalleryPhotosTable](/images/5-Workshop/5.3-backend-articles/GalleryPhotosTable.png)

**Mục đích:**  
Lưu metadata cho từng ảnh nằm trong bài viết để dùng cho Gallery/Explore.

**Primary Key:**  
- `photo_id` (HASH)

Metadata có thể bao gồm:  
- `articleId` — ảnh thuộc bài nào  
- đường dẫn S3  
- danh sách tag  
- thông tin vị trí chụp  

---

## 4. GalleryTrendsTable

![Cấu trúc GalleryTrendsTable](/images/5-Workshop/5.3-backend-articles/GalleryTrendsTable.png)

**Mục đích:**  
Lưu thống kê tần suất xuất hiện của từng tag.

**Primary Key:**  
- `tag_name` (HASH)

**Công dụng:**  
- Hiển thị các tag thịnh hành  
- Gợi ý bài viết theo chủ đề/địa danh  

---

## 5. UserProfilesTable

![Cấu trúc UserProfilesTable](/images/5-Workshop/5.3-backend-articles/UserProfilesTable.png)

**Mục đích:**  
Lưu thông tin hồ sơ công khai (avatar, họ tên, mô tả) để hiển thị trong feed và trang profile.

**Primary Key:**  
- `userId` (HASH)

Tách rời bảng này khỏi Cognito giúp hiển thị thông tin public mà không ảnh hưởng dữ liệu cá nhân.

---

## 6. LocationCacheTable

![Cấu trúc LocationCacheTable](/images/5-Workshop/5.3-backend-articles/LocationCacheTable.png)

**Mục đích:**  
Cache kết quả reverse-geocoding từ AWS Location để giảm chi phí và tăng tốc độ truy xuất.

**Primary Key:**  
- `geohash` (HASH)

**Thuộc tính:**  
- địa chỉ đã được chuyển đổi  
- `TTL` — tự động xoá theo thời gian  

---

# Tổng kết

Các bảng DynamoDB này tạo thành nền tảng dữ liệu của hệ thống, phục vụ:  
- CRUD bài viết  
- Danh sách yêu thích  
- Gallery + xu hướng tag  
- Profile người dùng  
- Cache vị trí tối ưu cho AWS Location  

Chúng được thiết kế để hoạt động tối ưu trong môi trường serverless với tính mở rộng cao.

