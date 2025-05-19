# Shipping API Documentation

This document provides comprehensive information about the Shipping API endpoints, including request and response formats, features, and examples.

## Table of Contents

- [Overview](#overview)
- [Environment URLs](#environment-urls)
- [Authentication](#authentication)
  - [Obtaining an Auth Token](#obtaining-an-auth-token)
  - [User Roles and Permissions](#user-roles-and-permissions)
  - [Authentication Headers](#authentication-headers)
- [Testing Mode](#testing-mode)
- [Endpoints](#endpoints)
  - [Get Available Carriers](#get-available-carriers)
  - [Get Shipping Rates](#get-shipping-rates)
  - [Calculate Duties](#calculate-duties)
  - [Create Shipping Label](#create-shipping-label)
  - [Track Shipment](#track-shipment)
- [Error Handling](#error-handling)

## Overview

The Shipping API provides a universal interface for shipping operations, including rate calculation, duties estimation, label creation, and shipment tracking. It supports multiple carriers (UPS, Bringer) through a single standardized API with user-specific customizations based on authentication.

## Environment URLs

The API is available in two environments:

### Production Environment
```
https://shippmate-server-d3d197bd152a.herokuapp.com
```

### Testing Environment(QA/DEV)
```
https://shippmate-test-server.herokuapp.com
```

## Authentication

### Obtaining an Auth Token

To obtain an authentication token, use the `/shipping/auth` endpoint:

**Endpoint:** `POST /shipping/auth`

**Request Body:**
```json
{
  "email": "your.email@example.com",
  "password": "your_password"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 14400,
    "user": {
      "_id": "user_id_12345"
    }
  }
}
```

The token is valid for 4 hours and should be included in the Authorization header for subsequent requests.


### Authentication Headers

All API endpoints support authentication using Bearer tokens:

```
Authorization: Bearer <your_token>
```

The token should contain user identification. Some endpoints support optional authentication to retrieve basic information without authentication.


## Testing Mode

The API supports a testing mode for label creation without processing actual payments.

To use testing mode:
1. Simply use the QA environment URL instead of the production URL
2. Rate and duties calculations will still provide accurate pricing

QA Environment: `https://shippmate-test-server.herokuapp.com`
Production Environment: `https://shippmate-server-d3d197bd152a.herokuapp.com`

## Endpoints

### Get Available Carriers

Retrieves information about available carriers and their capabilities.

**Endpoint:** `GET /shipping/carriers`
**Authentication:** Optional - authenticated users may see different carrier options

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "ups",
      "name": "UPS",
      "capabilities": ["domestic", "international", "duties", "pickup"],
      "services": [
        { "code": "01", "name": "UPS Next Day Air" },
        { "code": "02", "name": "UPS 2nd Day Air" },
        { "code": "03", "name": "UPS Ground" }
      ]
    },
    {
      "id": "bringer",
      "name": "Bringer",
      "capabilities": ["international", "duties"],
      "services": [
        { "code": "standard", "name": "Standard" },
        { "code": "express", "name": "Express" }
      ]
    }
  ]
}
```

### Get Shipping Rates

Calculates shipping rates from multiple carriers with a unified request format.

**Endpoint:** `POST /shipping/rate`
**Authentication:** Optional - authenticated users may receive custom rates and carriers

**Request Body:**

```json
{
  "from": {
    "name": "John Doe",
    "companyName": "Company Inc",
    "addressLine": "123 Main St",
    "addressLine2": "Apt 4B",
    "city": "New York",
    "state": "NY",
    "postalCode": "10001",
    "country": "US",
    "phone": "+12125551234"
  },
  "to": {
    "name": "Jane Smith",
    "companyName": "Receiving Corp",
    "addressLine": "456 Other Ave",
    "addressLine2": "Suite 7",
    "city": "Los Angeles",
    "state": "CA",
    "postalCode": "90001",
    "country": "US",
    "phone": "+13105551234"
  },
  "packages": [
    {
      "weight": "10",
      "weightUnit": "lb",
      "length": "12",
      "width": "10",
      "height": "8",
      "measurementUnit": "in"
    }
  ],
  "carrier": "all", 
  "service": {
    "code": "03", // Optional service code if you want rates for a specific service
    "name": "UPS Ground" // Optional
  },
  "shipmentDate": "2023-06-15", // Optional
  "declaredValue": 150, // Optional, for international shipments
  "currency": "USD", 
  "incoterms": "DDP" // Currently supporting only DDP 
}
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "carrier": "ups",
      "service": {
        "code": "03",
        "name": "UPS Ground"
      },
      "totalCharge": "21.24",
      "currency": "USD",
      "transitDays": "3",
      "deliveryDate": "2023-06-18T12:00:00.000Z"
    },
    {
      "carrier": "bringer",
      "service": {
        "code": "standard",
        "name": "Standard"
      },
      "totalCharge": "29.99",
      "currency": "USD",
      "transitDays": "5",
      "deliveryDate": "2023-06-20T12:00:00.000Z"
    }
  ],
  "count": 2
}
```

### Calculate Duties

Calculates duties and taxes for international shipments.

**Endpoint:** `POST /shipping/duties`
**Authentication:** Optional - works the same for all users

**Request Body:**

```json
{
  "from": {
    "country": "US"
  },
  "to": {
    "country": "CA"
  },
  "items": [
    {
      "id": "1",
      "quantity": "2",
      "hsCode": "610910",
      "contentDescription": "Cotton T-Shirt",
      "contentEstimatedValue": "25.00"
    },
    {
      "id": "2",
      "quantity": "1",
      "hsCode": "420222",
      "contentDescription": "Leather Handbag",
      "contentEstimatedValue": "120.00"
    }
  ],
  "declaredValue": 170.00,
  "currency": "USD",
  "incoterms": "DDP", // Currently supporting only DDP 
  "carrier": "ups" // Specify carrier to use for duties calculation (ups or bringer)
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "duties": 15.30,
    "taxes": 8.50,
    "brokerageFees": 35.62,
    "total": 59.42,
    "currency": "USD",
    "domestic": false
  }
}
```

### Create Shipping Label

Creates a shipping label with a carrier after calculating rates and duties.

**Endpoint:** `POST /shipping/label`
**Authentication:** Required - behavior varies based on user role

**Request Body:**

```json
{
  "from": {
    "name": "John Doe",
    "companyName": "Company Inc",
    "addressLine": "123 Main St",
    "addressLine2": "Apt 4B",
    "city": "New York",
    "state": "NY",
    "postalCode": "10001",
    "country": "US",
    "phone": "+12125551234"
  },
  "to": {
    "name": "Jane Smith",
    "companyName": "Receiving Corp",
    "addressLine": "456 Other Ave",
    "addressLine2": "Suite 7",
    "city": "Los Angeles",
    "state": "CA",
    "postalCode": "90001",
    "country": "US",
    "phone": "+13105551234"
  },
  "packages": [
    {
      "weight": "10",
      "weightUnit": "lb",
      "length": "12",
      "width": "10",
      "height": "8",
      "measurementUnit": "in",
      "packageType": {
        "code": "02",
        "description": "Package"
      }
    }
  ],
  "service": {
    "code": "03",
    "name": "UPS Ground"
  },
  "carrier": "ups",
  "shipmentDate": "2023-06-15",
  "items": [
    {
      "id": "1",
      "quantity": "2",
      "hsCode": "610910",
      "contentDescription": "Cotton T-Shirt",
      "contentEstimatedValue": "25.00"
    },
    {
      "id": "2",
      "quantity": "1",
      "hsCode": "420222",
      "contentDescription": "Leather Handbag",
      "contentEstimatedValue": "120.00"
    }
  ],
  "declaredValue": 170.00,
  "currency": "USD",
  "incoterms": "DDP", // Currently supporting only DDP 
  "shippingPurpose": "SALE", 
  "pickup": {  // UPS pickup object
    "pickupDate": "2023-06-16",
    "pickupStart": "09:00",
    "pickupEnd": "17:00",
    "pickupType": "01",
    "residential": "01"
  },
}
```

**Label Creation Process:**

1. Validate input parameters
2. Verify carrier is available for the user
3. Calculate duties (if international shipment)
4. Calculate total cost (shipping + duties)
5. Create shipping label with the carrier
6. Return label, tracking information, and cost

**Response:**

```json
{
  "success": true,
  "data": {
    "trackingNumber": "1Z999AA10123456784",
    "label": "data:image/gif;base64,R0lGODlhAQABAI...", // Base64 encoded label image
    "barcode": "data:image/png;base64,iVBORw0KG...", // Base64 encoded barcode image
    "carrier": "ups",
    "service": {
      "code": "03",
      "name": "UPS Ground"
    },
    "estimatedDelivery": "2023-06-18T12:00:00.000Z",
    "costs": {
      "shipping": 21.24,
      "duties": 59.42,
      "pickup": 5.00,
      "total": 85.66,
      "currency": "USD"
    }
  },
  "message": "Label created successfully with UPS"
}
```

### Track Shipment

Tracks a shipment with a unified interface supporting multiple carriers.

**Endpoint:** `POST /shipping/tracking`
**Authentication:** Optional - works the same for all users

**Request Body:**

```json
{
  "trackingNumber": "1Z999AA10123456784",
  "carrier": "ups",
  "originCountry": "US"
}
```

**Response:**

The response format is standardized across all carriers to provide a consistent experience:

```json
{
  "success": true,
  "data": {
    "trackingNumber": "1Z999AA10123456784",
    "deliveryDate": [], // Array of potential delivery dates
    "activity": [
      {
        "location": {
          "address": {
            "city": "New York",
            "stateProvince": "NY",
            "countryCode": "US",
            "country": "US"
          }
        },
        "status": {
          "type": "I",
          "description": "Arrived at Facility",
          "code": "AR",
          "statusCode": "005"
        },
        "date": "20230615",
        "time": "143000"
      }
    ],
    "currentStatus": {
      "description": "In Transit",
      "code": "005",
      "simplifiedTextDescription": "Package is in transit to the destination"
    },
    "packageAddress": [
      {
        "type": "ORIGIN",
        "address": {
          "city": "New York",
          "stateProvince": "NY",
          "countryCode": "US",
          "country": "US"
        }
      },
      {
        "type": "DESTINATION",
        "address": {
          "city": "Los Angeles",
          "stateProvince": "CA",
          "countryCode": "US",
          "country": "US"
        }
      }
    ],
    "weight": {
      "unitOfMeasurement": "LBS",
      "weight": "10.00"
    },
    "service": {
      "code": "03",
      "levelCode": "",
      "description": "UPS Ground"
    },
    "referenceNumber": [
      {
        "type": "SHIPMENT",
        "number": "1Z999AA10123456784"
      }
    ],
    "dimension": {
      "height": "8.00",
      "length": "12.00",
      "width": "10.00",
      "unitOfDimension": "IN"
    },
    "isSmartPackage": false,
    "packageCount": 1
  },
  "carrier": "ups"
}
```

## Error Handling

Errors are returned with appropriate HTTP status codes and descriptive messages:

```json
{
  "success": false,
  "message": "Failed to calculate shipping rates",
  "error": "Carrier service unavailable"
}
```

Common status codes:
- `400`: Bad Request - Invalid input parameters
- `401`: Unauthorized - Authentication required
- `404`: Not Found - Resource not found
- `500`: Internal Server Error - Server-side error

For validation errors, the response will include specific information about the missing or invalid fields. 