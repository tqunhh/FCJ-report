---
title: " Backend – Dịch vụ Bài viết "
date: 2025-12-08
weight: 1
chapter: false
pre : " <b> 5.3.1. </b> "
---

#### Cấu trúc Template Code của các chức năng

![Function Template 1](/images/5-Workshop/5.3-backend-articles/FUNCTION1.PNG)

![Function Template 2](/images/5-Workshop/5.3-backend-articles/FUNCTION2.PNG)

#### Tổng quan các Hàm Lambda

![Toàn bộ Lambda Functions](/images/5-Workshop/5.3-backend-articles/Full_lamdba_Function.png)

#### Tích hợp API Gateway

![Toàn bộ API Gateway](/images/5-Workshop/5.3-backend-articles/Full_API_GATEWAY.png)

---

## Tóm tắt các Hàm Lambda

Phần này cung cấp một cái nhìn tổng quan súc tích về tất cả các hàm Lambda được sử dụng trong các module Bài viết, Hồ sơ và Mục yêu thích. Các hàm này cùng nhau tạo nên luồng công việc backend của hệ thống nội dung.

## 1. Các Hàm Bài viết (CRUD + Tìm kiếm + Tải lên)

- **CreateArticleFunction**
  - Endpoint: `POST /articles` (Yêu cầu xác thực)
  - Vai trò: Tạo một bài viết mới, xử lý siêu dữ liệu hình ảnh, giải quyết vị trí địa lý bằng AWS Location, lưu dữ liệu vào DynamoDB và S3.

- **GetArticleFunction**
  - Endpoint: `GET /articles/{articleId}` (Công khai)
  - Vai trò: Truy xuất một bài viết và hình ảnh duy nhất đồng thời áp dụng các quy tắc hiển thị.

- **ListArticlesFunction**
  - Endpoint: `GET /articles` (Yêu cầu xác thực)
  - Vai trò: Liệt kê các bài viết công khai được làm giàu thêm dữ liệu hồ sơ và siêu dữ liệu hình ảnh.

- **UpdateArticleFunction**
  - Endpoint: `PATCH /articles/{articleId}`
  - Vai trò: Xác minh quyền sở hữu và cập nhật nội dung bài viết, khả năng hiển thị, vị trí và hình ảnh.

- **DeleteArticleFunction**
  - Endpoint: `DELETE /articles/{articleId}`
  - Vai trò: Xóa vĩnh viễn bài viết và các hình ảnh liên quan khỏi S3.

- **SearchArticlesFunction**
  - Endpoint: `GET /search` (Công khai)
  - Vai trò: Thực hiện tìm kiếm từ khóa/thẻ bằng cách sử dụng các truy vấn DynamoDB.

- **GetUploadUrlFunction**
  - Endpoint: `POST /upload-url` (Yêu cầu xác thực)
  - Vai trò: Tạo một URL được ký trước để tải hình ảnh trực tiếp lên S3.

## 2. Hàm Hồ sơ Người dùng

- **GetUserArticlesFunction**
  - Endpoint: `GET /users/{userId}/articles`
  - Vai trò: Truy xuất tất cả các bài viết công khai được tạo bởi một người dùng cụ thể cho trang hồ sơ của họ.

## 3. Các Hàm Mục yêu thích

- **FavoriteArticleFunction**
  - Endpoint: `POST /articles/{articleId}/favorite`
  - Vai trò: Thêm ánh xạ `(userId, articleId)` vào `UserFavoritesTable`.

- **UnfavoriteArticleFunction**
  - Endpoint: `DELETE /articles/{articleId}/favorite`
  - Vai trò: Xóa một bản ghi yêu thích.

- **ListFavoriteArticlesFunction**
  - Endpoint: `GET /me/favorites`
  - Vai trò: Trả về tất cả các bài viết được người dùng đã xác thực đánh dấu là yêu thích.

## Tóm tắt chung

Các Hàm Lambda này tích hợp DynamoDB, S3, Cognito, và AWS Location để tạo ra một backend có tính mô-đun, có thể mở rộng, cung cấp sức mạnh cho hệ thống quản lý bài viết.