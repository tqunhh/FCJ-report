---
title: "Cấu hình AWS Cognito"
date: 2025-12-08
weight: 1
chapter: false
pre: "<b>5.5.1. </b>"
---

## AWS Cognito - Dịch vụ Xác thực Người dùng

### Giới thiệu về AWS Cognito

Amazon Cognito là dịch vụ quản lý danh tính (Identity Management) được xây dựng để giúp ứng dụng của bạn có khả năng đăng ký, đăng nhập, xác thực người dùng mà không phải tự thiết kế hệ thống authentication phức tạp. Cognito hỗ trợ mở rộng theo chiều ngang, độ sẵn sàng cao và an toàn theo tiêu chuẩn AWS Security.

Thông qua Cognito User Pool, các ứng dụng có thể xác thực người dùng bằng email, số điện thoại, mật khẩu hoặc các nhà cung cấp đăng nhập xã hội (Google, Facebook, Apple). Bên cạnh đó, Cognito phát hành các mã token (JWT) để ứng dụng Frontend và Backend có thể kiểm soát quyền truy cập mà không cần lưu trạng thái session trên server.

**User Pool** chịu trách nhiệm:
- Quản lý danh tính người dùng
- Xác thực OTP
- Lưu trữ profile người dùng
- Tạo tokens

**App Client và Hosted UI** hỗ trợ giao diện đăng nhập tích hợp, phục vụ tốt cho ứng dụng Web và Mobile mà không cần tự xây dựng màn hình login từ đầu.

### Tổng quan Workshop

Trong workshop này, bạn sẽ triển khai **một hệ thống xác thực hoàn chỉnh** dựa trên Cognito, mô phỏng đầy đủ các thành phần thường gặp trong môi trường doanh nghiệp.

Chúng ta sẽ sử dụng **hai nhóm môi trường ứng dụng**:

#### "Frontend App"
Đây là ứng dụng React mô phỏng phần giao diện người dùng (Client Application). Ứng dụng này sẽ tích hợp Hosted UI của Cognito và quản lý id_token, access_token cũng như refresh_token.

Frontend sẽ được cấu hình để:
- Kết nối với Cognito Domain
- Nhận token sau khi người dùng đăng nhập
- Gọi API bảo vệ bằng Access Token

#### "Backend API"
Đây là phần mô phỏng tầng dịch vụ (Service Layer) của doanh nghiệp. Backend bao gồm API Gateway, Cognito Authorizer và hàm Lambda dùng để xử lý logic.

Các API trong môi trường này sẽ:
- Yêu cầu access_token hợp lệ để truy cập
- Xác thực token bằng Cognito trước khi chuyển tiếp đến Lambda
- Đọc thông tin người dùng từ JWT
- Mô phỏng hành vi xác thực & phân quyền của các hệ thống backend production

---

## Bước 1: Tạo User Pool

Truy cập AWS Console → Cognito → User Pools → Create user pool

**Cấu hình:**
- Tên pool: `TravelGuideUserPool-staging`
- Sign-in options: Email
- Attributes: email, name

![Cấu hình User Pool](/images/5-Workshop/5.5-Policy/Setup_UserPool.png)

---

## Bước 2: Password Policy

Cấu hình yêu cầu mật khẩu:
- Có ít nhất 1 số
- Có ít nhất 1 chữ thường
- Có ít nhất 1 chữ viết hoa

![Password Policy](/images/5-Workshop/5.5-Policy/SetUp_PassWord_Policy.png)

**Tại sao cần những yêu cầu này?**
- Đảm bảo mật khẩu mạnh
- Đáp ứng tiêu chuẩn bảo mật
- Bảo vệ chống tấn công brute force

---

## Bước 3: Tạo App Client

Truy cập: App integration → App clients → Create app client

**Các bước:**
1. Nhập tên app client
2. Chọn Authentication flows:
   - `ALLOW_USER_PASSWORD_AUTH`
   - `ALLOW_REFRESH_TOKEN_AUTH`
   - `ALLOW_USER_SRP_AUTH`
3. Tạo app client

![Cấu hình App Client](/images/5-Workshop/5.5-Policy/Setup_AppClient.png)

![Authentication Flows](/images/5-Workshop/5.5-Policy/SetUp_AppClient_2.png)

**Giải thích Authentication Flows:**
- **USER_PASSWORD_AUTH**: Xác thực trực tiếp bằng username/password
- **REFRESH_TOKEN_AUTH**: Khả năng làm mới token
- **USER_SRP_AUTH**: Giao thức Secure Remote Password (khuyến nghị)

---

## Bước 4: Tạo User Test

Truy cập: User Pool → Users → Create user

**Thông tin User:**
- Username
- Địa chỉ email
- Số điện thoại (tùy chọn)
- Mật khẩu tạm thời

![Tạo Test User](/images/5-Workshop/5.5-Policy/CreateUserTest.png)

![User đã tạo](/images/5-Workshop/5.5-Policy/CreateUserTest2.png)

**Lưu ý:** User sẽ cần đổi mật khẩu khi đăng nhập lần đầu.

---

## Bước 5: Luồng Sign Up

Cognito tự động xử lý OTP - backend không cần code!

**Quy trình Đăng ký:**
1. Người dùng nhập email và mật khẩu
2. Cognito gửi mã xác thực đến email
3. Người dùng nhập mã để xác minh
4. Tài khoản được kích hoạt

<!-- Placeholder ảnh: Form đăng ký -->
![Form Đăng ký](#)

<!-- Placeholder ảnh: Xác thực email -->
![Xác thực Email](#)

**Lợi ích:**
- ✅ Không cần dịch vụ email tùy chỉnh
- ✅ Rate limiting tích hợp sẵn
- ✅ Logic retry tự động
- ✅ Tạo mã bảo mật

---

## Quản lý Token

### Giải thích JWT Tokens

Cognito phát hành ba loại tokens:

#### 1. ID Token
- Chứa thông tin danh tính người dùng
- Frontend dùng để hiển thị dữ liệu user
- Thời gian sống ngắn (mặc định 1 giờ)

```json
{
  "sub": "user-uuid",
  "email": "user@example.com",
  "name": "Nguyễn Văn A",
  "cognito:username": "nguyenvana"
}
```

#### 2. Access Token
- Dùng để ủy quyền API requests
- Gửi trong Authorization header
- Được xác thực bởi API Gateway

```
Authorization: Bearer <access_token>
```

#### 3. Refresh Token
- Dùng để lấy ID và Access tokens mới
- Thời gian sống dài (mặc định 30 ngày)
- Lưu trữ an toàn trên client

### Luồng Token

```
1. Người dùng Đăng nhập
   ↓
2. Cognito xác thực credentials
   ↓
3. Cognito phát hành tokens
   ↓
4. Frontend lưu trữ tokens
   ↓
5. Frontend gọi API với Access Token
   ↓
6. API Gateway xác thực token
   ↓
7. Lambda xử lý request
```

---

## Tích hợp Frontend

### Ví dụ React

```javascript
import { CognitoUserPool, CognitoUser, AuthenticationDetails } from 'amazon-cognito-identity-js';

const poolData = {
  UserPoolId: 'us-east-1_XXXXXXXXX',
  ClientId: 'your-app-client-id'
};

const userPool = new CognitoUserPool(poolData);

// Đăng nhập
function signIn(username, password) {
  const authenticationData = {
    Username: username,
    Password: password,
  };
  
  const authenticationDetails = new AuthenticationDetails(authenticationData);
  
  const userData = {
    Username: username,
    Pool: userPool
  };
  
  const cognitoUser = new CognitoUser(userData);
  
  cognitoUser.authenticateUser(authenticationDetails, {
    onSuccess: (result) => {
      const accessToken = result.getAccessToken().getJwtToken();
      const idToken = result.getIdToken().getJwtToken();
      const refreshToken = result.getRefreshToken().getToken();
      
      // Lưu tokens
      localStorage.setItem('accessToken', accessToken);
      localStorage.setItem('idToken', idToken);
      localStorage.setItem('refreshToken', refreshToken);
    },
    onFailure: (err) => {
      console.error(err);
    }
  });
}
```

---

## Tích hợp Backend

### API Gateway Cognito Authorizer

**Cấu hình:**
1. Tạo Cognito User Pool Authorizer
2. Chỉ định User Pool ID
3. Token source: `Authorization` header
4. Gắn vào API routes

**Lambda nhận thông tin user:**

```python
def lambda_handler(event, context):
    # Thông tin user từ Cognito
    claims = event['requestContext']['authorizer']['claims']
    
    user_id = claims['sub']
    email = claims['email']
    username = claims['cognito:username']
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'Xin chào {username}!',
            'userId': user_id
        })
    }
```

---

## Điểm chính

1. **Cognito User Pool** quản lý danh tính người dùng
2. **JWT Tokens** cung cấp xác thực stateless
3. **Hosted UI** đơn giản hóa tích hợp frontend
4. **Cognito Authorizer** bảo mật API Gateway
5. **Không cần quản lý session** trên backend
6. **Có khả năng mở rộng** và **sẵn sàng cao** mặc định

---

## Best Practices

1. **Dùng SRP authentication** để tăng cường bảo mật
2. **Bật MFA** cho các thao tác nhạy cảm
3. **Rotate refresh tokens** thường xuyên
4. **Lưu trữ tokens an toàn** (không dùng localStorage cho production)
5. **Implement logic refresh token**
6. **Dùng HTTPS** cho mọi giao tiếp
7. **Đặt thời gian hết hạn token** phù hợp
