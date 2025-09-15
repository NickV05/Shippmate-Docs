# buy.shippmate Authentication Usage Examples / Примеры использования аутентификации buy.shippmate

[English](#english) | [Русский](#russian)

---

<a name="russian"></a>
## 🇷🇺 Русская версия

### Обзор
Этот документ содержит примеры использования новых методов аутентификации buy.shippmate.

### API Эндпоинты

#### 1. Регистрация
```http
POST /shipping/auth/signup
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123",
  "firstName": "Иван",
  "lastName": "Иванов",
  "phone": "+79123456789"  // необязательно
}
```

**Ответ:**
```json
{
  "success": true,
  "authToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "message": "Account created successfully"
}
```

#### 2. Вход
```http
POST /shipping/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "SecurePass123"
}
```

**Ответ:**
```json
{
  "success": true,
  "authToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "email": "user@example.com",
    "firstName": "Иван",
    "lastName": "Иванов",
    "phone": "+79123456789"
  }
}
```

#### 3. Выход
```http
POST /shipping/auth/logout
```

**Ответ:**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

### Обработка ошибок

#### Ошибки регистрации:
- **Email уже существует**: `"User already exists."`
- **Неверный формат email**: `"Provide a valid email address."`
- **Слабый пароль**: `"Password must have at least 6 characters and contain at least one number, one lowercase and one uppercase letter."`
- **Отсутствуют обязательные поля**: `"All fields are required."`

#### Ошибки входа:
- **Пользователь не найден**: `"User not found."`
- **Неверный пароль**: `"Unable to authenticate the user"`
- **Отсутствуют учетные данные**: `"Email and password are required."`

### Конфигурация Cookie

Система использует cookie для всего домена `.shippmate.com`, что позволяет:
- Автоматическую аутентификацию на всех поддоменах
- Автоматическое обновление при каждом запросе
- Срок действия 12 часов

---

<a name="english"></a>
## 🇬🇧 English Version

### Overview
This document provides examples of how to use the newly implemented buy.shippmate authentication methods.

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


1. **Cookie Properties**:
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

2. **Cookie Structure**:
   ```json
   {
     "token": "JWT_TOKEN_HERE",
     "userId": "USER_ID",
     "expiresAt": 1234567890000
   }
   ```



