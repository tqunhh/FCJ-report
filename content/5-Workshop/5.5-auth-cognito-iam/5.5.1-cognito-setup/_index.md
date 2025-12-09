---
title: "AWS Cognito Setup"
date: 2025-12-08
weight: 1
chapter: false
pre: "<b>5.5.1. </b>"
---

## AWS Cognito - User Authentication Service

### Introduction to AWS Cognito

Amazon Cognito is an Identity Management service built to help your application handle user registration, login, and authentication without designing a complex authentication system from scratch. Cognito supports horizontal scaling, high availability, and security according to AWS Security standards.

Through Cognito User Pool, applications can authenticate users using email, phone number, password, or social login providers (Google, Facebook, Apple). Additionally, Cognito issues JWT tokens so Frontend and Backend applications can control access without storing session state on the server.

**User Pool** is responsible for:
- Managing user identities
- OTP authentication
- Storing user profiles
- Generating tokens

**App Client and Hosted UI** provide integrated login interface, serving Web and Mobile applications well without building login screens from scratch.

### Workshop Overview

In this workshop, you will deploy **a complete authentication system** based on Cognito, simulating all components commonly found in enterprise environments.

We will use **two application environment groups**:

#### "Frontend App"
This is a React application simulating the user interface (Client Application). This application will integrate Cognito's Hosted UI and manage id_token, access_token, and refresh_token.

Frontend will be configured to:
- Connect to Cognito Domain
- Receive tokens after user login
- Call protected APIs using Access Token

#### "Backend API"
This simulates the enterprise Service Layer. Backend includes API Gateway, Cognito Authorizer, and Lambda functions for processing logic.

APIs in this environment will:
- Require valid access_token for access
- Validate tokens using Cognito before forwarding to Lambda
- Read user information from JWT
- Simulate authentication & authorization behavior of production backend systems

---

## Step 1: Create User Pool

Navigate to AWS Console → Cognito → User Pools → Create user pool

**Configuration:**
- Pool name: `TravelGuideUserPool-staging`
- Sign-in options: Email
- Attributes: email, name

![User Pool Configuration](/images/5-Workshop/5.5-Policy/Setup_UserPool.png)

---

## Step 2: Password Policy

Configure password requirements:
- At least 1 number
- At least 1 lowercase letter
- At least 1 uppercase letter

![Password Policy](/images/5-Workshop/5.5-Policy/SetUp_PassWord_Policy.png)

**Why these requirements?**
- Ensures strong passwords
- Meets security compliance standards
- Protects against brute force attacks

---

## Step 3: Create App Client

Navigate to: App integration → App clients → Create app client

**Steps:**
1. Enter app client name
2. Select Authentication flows:
   - `ALLOW_USER_PASSWORD_AUTH`
   - `ALLOW_REFRESH_TOKEN_AUTH`
   - `ALLOW_USER_SRP_AUTH`
3. Create app client

![App Client Configuration](/images/5-Workshop/5.5-Policy/Setup_AppClient.png)

![Authentication Flows](/images/5-Workshop/5.5-Policy/SetUp_AppClient_2.png)

**Authentication Flows Explained:**
- **USER_PASSWORD_AUTH**: Direct username/password authentication
- **REFRESH_TOKEN_AUTH**: Token refresh capability
- **USER_SRP_AUTH**: Secure Remote Password protocol (recommended)

---

## Step 4: Create Test User

Navigate to: User Pool → Users → Create user

**User Information:**
- Username
- Email address
- Phone number (optional)
- Temporary password

![Create Test User](/images/5-Workshop/5.5-Policy/CreateUserTest.png)

![User Created](/images/5-Workshop/5.5-Policy/CreateUserTest2.png)

**Note:** User will need to change password on first login.

---

## Step 5: Sign Up Flow

Cognito automatically handles OTP verification - no backend code needed!

**Sign Up Process:**
1. User enters email and password
2. Cognito sends verification code to email
3. User enters code to verify
4. Account is activated

<!-- Image placeholder: Sign up form -->
![Sign Up Form](#)

<!-- Image placeholder: Email verification -->
![Email Verification](#)

**Benefits:**
- ✅ No custom email service needed
- ✅ Built-in rate limiting
- ✅ Automatic retry logic
- ✅ Secure code generation

---

## Token Management

### JWT Tokens Explained

Cognito issues three types of tokens:

#### 1. ID Token
- Contains user identity information
- Used by frontend to display user data
- Short-lived (1 hour default)

```json
{
  "sub": "user-uuid",
  "email": "user@example.com",
  "name": "John Doe",
  "cognito:username": "johndoe"
}
```

#### 2. Access Token
- Used to authorize API requests
- Sent in Authorization header
- Validated by API Gateway

```
Authorization: Bearer <access_token>
```

#### 3. Refresh Token
- Used to obtain new ID and Access tokens
- Long-lived (30 days default)
- Stored securely on client

### Token Flow

```
1. User Login
   ↓
2. Cognito validates credentials
   ↓
3. Cognito issues tokens
   ↓
4. Frontend stores tokens
   ↓
5. Frontend calls API with Access Token
   ↓
6. API Gateway validates token
   ↓
7. Lambda processes request
```

---

## Frontend Integration

### React Example

```javascript
import { CognitoUserPool, CognitoUser, AuthenticationDetails } from 'amazon-cognito-identity-js';

const poolData = {
  UserPoolId: 'us-east-1_XXXXXXXXX',
  ClientId: 'your-app-client-id'
};

const userPool = new CognitoUserPool(poolData);

// Sign In
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
      
      // Store tokens
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

## Backend Integration

### API Gateway Cognito Authorizer

**Configuration:**
1. Create Cognito User Pool Authorizer
2. Specify User Pool ID
3. Token source: `Authorization` header
4. Attach to API routes

**Lambda receives user info:**

```python
def lambda_handler(event, context):
    # User info from Cognito
    claims = event['requestContext']['authorizer']['claims']
    
    user_id = claims['sub']
    email = claims['email']
    username = claims['cognito:username']
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'Hello {username}!',
            'userId': user_id
        })
    }
```

---

## Key Takeaways

1. **Cognito User Pool** manages user identities
2. **JWT Tokens** provide stateless authentication
3. **Hosted UI** simplifies frontend integration
4. **Cognito Authorizer** secures API Gateway
5. **No session management** needed on backend
6. **Scalable** and **highly available** by default

---

## Best Practices

1. **Use SRP authentication** for enhanced security
2. **Enable MFA** for sensitive operations
3. **Rotate refresh tokens** regularly
4. **Store tokens securely** (not in localStorage for production)
5. **Implement token refresh** logic
6. **Use HTTPS** for all communications
7. **Set appropriate token expiration** times
