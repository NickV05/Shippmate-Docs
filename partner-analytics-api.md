# Partner Analytics API

API for analytics collection partners to submit shipping events to GA4.

## Authentication

All requests require JWT token in the `Authorization` header:
```
Authorization: Bearer <your_jwt_token>
```

User must have `ANALYTICS_COLLECTION_PARTNER` partner type enabled.

## Endpoints

### POST /api/partner-analytics/rate-view

Track when users view shipping rates.

**Request Body:**
```json
{
  "warehouse": "Oakland Park",
  "items": [
    {
      "serviceDescription": "UPS Ground",
      "cost": 15.99,
      "carrier": "UPS",
      "from": { "country": "US", "state": "FL", "city": "Miami" },
      "to": { "country": "US", "state": "NY", "city": "New York" },
      "package": {
        "weight": "2.5",
        "weightUnit": "LBS",
        "contentType": "package"
      },
      "isDomestic": true,
      "isCommercial": false,
      "incoterms": "DAP"
    }
  ],
  "userId": "customer-123",
  "clientId": "GA1.1.123456789.1700000000"
}
```

### POST /api/partner-analytics/purchase

Track completed purchases.

**Request Body:**
```json
{
  "warehouse": "Oakland Park",
  "orderId": "ORD-12345",
  "serviceDescription": "UPS Ground",
  "totalCost": 25.99,
  "carrier": "UPS",
  "from": { "country": "US", "state": "FL", "city": "Miami" },
  "to": { "country": "US", "state": "NY", "city": "New York" },
  "package": {
    "weight": "2.5",
    "weightUnit": "LBS",
    "contentType": "package"
  },
  "isDomestic": true,
  "isCommercial": false,
  "hasPickup": false,
  "hasDuties": false,
  "incoterms": "DAP",
  "paymentType": "card",
  "shippingCost": 20.00,
  "dutiesCost": 5.99,
  "userId": "customer-123",
  "clientId": "GA1.1.123456789.1700000000"
}
```

## Tracking Fields (Optional but Strongly Recommended)

| Field | Description |
|-------|-------------|
| `clientId` | GA4 cookie ID from web (e.g., `GA1.1.xxxxx.xxxxx` from `_ga` cookie) |
| `userId` | Your customer's ID (end-user identifier) |

> **Important:** While these fields are optional, we **strongly recommend** providing them to enable accurate user behavior analysis and cross-session tracking in GA4. If not provided, random values will be generated, which limits the ability to track user journeys across multiple events.

## Response

**Success (200):**
```json
{ "success": true, "message": "Event tracked successfully" }
```

**Error (400/401/403):**
```json
{ "success": false, "error": "Error message", "details": [...] }
```

---

# API Партнерской Аналитики

API для партнеров по сбору аналитики для отправки событий доставки в GA4.

## Аутентификация

Все запросы требуют JWT токен в заголовке `Authorization`:
```
Authorization: Bearer <ваш_jwt_токен>
```

У пользователя должен быть включен тип партнера `ANALYTICS_COLLECTION_PARTNER`.

## Эндпоинты

### POST /api/partner-analytics/rate-view

Отслеживание просмотра тарифов доставки.

**Тело запроса:**
```json
{
  "warehouse": "Oakland Park",
  "items": [
    {
      "serviceDescription": "UPS Ground",
      "cost": 15.99,
      "carrier": "UPS",
      "from": { "country": "US", "state": "FL", "city": "Miami" },
      "to": { "country": "US", "state": "NY", "city": "New York" },
      "package": {
        "weight": "2.5",
        "weightUnit": "LBS",
        "contentType": "package"
      },
      "isDomestic": true,
      "isCommercial": false,
      "incoterms": "DAP"
    }
  ],
  "userId": "customer-123",
  "clientId": "GA1.1.123456789.1700000000"
}
```

### POST /api/partner-analytics/purchase

Отслеживание завершенных покупок.

**Тело запроса:**
```json
{
  "warehouse": "Oakland Park",
  "orderId": "ORD-12345",
  "serviceDescription": "UPS Ground",
  "totalCost": 25.99,
  "carrier": "UPS",
  "from": { "country": "US", "state": "FL", "city": "Miami" },
  "to": { "country": "US", "state": "NY", "city": "New York" },
  "package": {
    "weight": "2.5",
    "weightUnit": "LBS",
    "contentType": "package"
  },
  "isDomestic": true,
  "isCommercial": false,
  "hasPickup": false,
  "hasDuties": false,
  "incoterms": "DAP",
  "paymentType": "card",
  "shippingCost": 20.00,
  "dutiesCost": 5.99,
  "userId": "customer-123",
  "clientId": "GA1.1.123456789.1700000000"
}
```

## Поля отслеживания (Опционально, но настоятельно рекомендуется)

| Поле | Описание |
|------|----------|
| `clientId` | GA4 cookie ID с веба (например, `GA1.1.xxxxx.xxxxx` из cookie `_ga`) |
| `userId` | ID вашего клиента (идентификатор конечного пользователя) |

> **Важно:** Хотя эти поля являются опциональными, мы **настоятельно рекомендуем** их предоставлять для обеспечения точного анализа поведения пользователей и отслеживания между сессиями в GA4. Если значения не предоставлены, будут сгенерированы случайные значения, что ограничивает возможность отслеживания пути пользователя между событиями.

## Ответ

**Успех (200):**
```json
{ "success": true, "message": "Event tracked successfully" }
```

**Ошибка (400/401/403):**
```json
{ "success": false, "error": "Сообщение об ошибке", "details": [...] }
```
