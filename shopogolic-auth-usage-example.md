# Shopogolic Authentication Usage Examples

## Overview
This document provides examples of how to use the newly implemented Shopogolic authentication methods.

## API Endpoints

### 1. Signup
```http
POST /shipping/auth/signup
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123",
  "firstName": "John",
  "lastName": "Doe",
  "phone": "+1234567890"  // optional
}
```

**Response:**
```json
{
  "success": true,
  "authToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "message": "Account created successfully"
}
```

**Cookie Set:**
```
shopogolic_auth={
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "userId": "507f1f77bcf86cd799439011",
  "expiresAt": 1703721600000
}
```

### 2. Login
```http
POST /shipping/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123"
}
```

**Response:**
```json
{
  "success": true,
  "authToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "phone": "+1234567890"
  }
}
```

### 3. Logout
```http
POST /shipping/auth/logout
```

**Response:**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

## Error Handling & Validation

### Signup Validation

1. **Email Already Exists**
```http
POST /shipping/auth/signup
```
Response (400):
```json
{
  "success": false,
  "message": "User already exists."
}
```

2. **Invalid Email Format**
Response (400):
```json
{
  "success": false,
  "message": "Provide a valid email address."
}
```

3. **Weak Password**
Response (400):
```json
{
  "success": false,
  "message": "Password must have at least 6 characters and contain at least one number, one lowercase and one uppercase letter."
}
```

4. **Missing Required Fields**
Response (400):
```json
{
  "success": false,
  "message": "All fields are required."
}
```

### Login Validation

1. **User Not Found**
```http
POST /shipping/auth/login
```
Response (401):
```json
{
  "success": false,
  "message": "User not found."
}
```

2. **Incorrect Password**
Response (401):
```json
{
  "success": false,
  "message": "Unable to authenticate the user"
}
```

3. **Missing Credentials**
Response (400):
```json
{
  "success": false,
  "message": "Email and password are required."
}
```

## Domain Cookie Configuration

### How Domain Cookies Work

The authentication system uses domain-wide cookies that work across all ShippMate subdomains:

1. **Cookie Domain**: `.shippmate.com`
   - The leading dot (.) makes the cookie available to all subdomains
   - Works on: `shippmate.com`, `buy.shippmate.com`, `api.shippmate.com`, etc.

2. **Cookie Properties**:
   ```javascript
   {
     httpOnly: true,        // Prevents JavaScript access (XSS protection)
     secure: true,          // HTTPS only in production
     sameSite: 'lax',      // CSRF protection
     domain: '.shippmate.com',
     path: '/',            // Available on all paths
     maxAge: 43200000      // 12 hours
   }
   ```

3. **Cookie Structure**:
   ```json
   {
     "token": "JWT_TOKEN_HERE",
     "userId": "USER_ID",
     "expiresAt": 1234567890000
   }
   ```

