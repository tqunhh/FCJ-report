---
title: " Backend – Article Service "
date: 2025-12-08
weight: 1
chapter: false
pre: "<b>5.3.1.  </b>"
---

#### Template Code Structure

![Function Template 1](/images/5-Workshop/5.3-backend-articles/FUNCTION1.PNG)

![Function Template 2](/images/5-Workshop/5.3-backend-articles/FUNCTION2.PNG)

#### Lambda Functions Overview

![Full Lambda Functions](/images/5-Workshop/5.3-backend-articles/Full_lamdba_Function.png)

#### API Gateway Integration

![Full API Gateway](/images/5-Workshop/5.3-backend-articles/Full_API_GATEWAY.png)

---

## Summary of Lambda Functions

This section provides a concise overview of all Lambda functions used in the Article, Profile, and Favorites modules. These functions collectively form the backend workflow of the content system.

## 1. Article Functions (CRUD + Search + Upload)

- **CreateArticleFunction**
	- Endpoint: `POST /articles` (Auth required)
	- Role: Creates a new article, processes image metadata, resolves geolocation using AWS Location, saves data into DynamoDB and S3.

- **GetArticleFunction**
	- Endpoint: `GET /articles/{articleId}` (Public)
	- Role: Retrieves a single article and images while applying visibility rules.

- **ListArticlesFunction**
	- Endpoint: `GET /articles` (Auth required)
	- Role: Lists public articles enriched with profile data and image metadata.

- **UpdateArticleFunction**
	- Endpoint: `PATCH /articles/{articleId}`
	- Role: Verifies ownership and updates article content, visibility, location, and images.

- **DeleteArticleFunction**
	- Endpoint: `DELETE /articles/{articleId}`
	- Role: Permanently deletes the article and related images from S3.

- **SearchArticlesFunction**
	- Endpoint: `GET /search` (Public)
	- Role: Performs keyword/tag search using DynamoDB queries.

- **GetUploadUrlFunction**
	- Endpoint: `POST /upload-url` (Auth required)
	- Role: Generates a presigned URL for uploading images directly to S3.

## 2. User Profile Function

- **GetUserArticlesFunction**
	- Endpoint: `GET /users/{userId}/articles`
	- Role: Retrieves all public articles created by a specific user for their profile page.

## 3. Favorites Functions

- **FavoriteArticleFunction**
	- Endpoint: `POST /articles/{articleId}/favorite`
	- Role: Adds `(userId, articleId)` mapping into `UserFavoritesTable`.

- **UnfavoriteArticleFunction**
	- Endpoint: `DELETE /articles/{articleId}/favorite`
	- Role: Removes a favorite record.

- **ListFavoriteArticlesFunction**
	- Endpoint: `GET /me/favorites`
	- Role: Returns all articles favorited by the authenticated user.

## Overall Summary

These Lambda functions integrate DynamoDB, S3, Cognito, and AWS Location to create a modular, scalable backend powering the article management system.