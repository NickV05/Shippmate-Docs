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
  - [Get Shipping Limits](#get-shipping-limits)
  - [Validate Package Dimensions](#validate-package-dimensions)
  - [Create Pickup Request](#create-pickup-request)
- [Pickup Feature Best Practices](#pickup-feature-best-practices)
- [Shipping Limits](#shipping-limits)
  - [Bringer Limits by Destination](#bringer-limits-by-destination)
  - [UPS Limits](#ups-limits)
  - [Value Limits](#value-limits)
- [Error Handling](#error-handling)
- [Country-Specific Requirements](#country-specific-requirements)

## Overview

The Shipping API provides a universal interface for shipping operations, including rate calculation, duties estimation, label creation, and shipment tracking. It supports multiple carriers (UPS, Bringer) through a single standardized API with user-specific customizations based on authentication.

## Environment URLs

The API is available in two environments:

### Production Environment

```
https://shippmate-server-d3d197bd152a.herokuapp.com
```

### Testing Environment (QA/DEV)

```
https://shippmate-server-test-976f52a5a04a.herokuapp.com
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

### User Roles and Permissions

The API supports different user types with varying permissions:

1. **Regular Users**: Access to all carriers with standard rates
2. **Admin Users**: Administrative access with additional features
3. **Users with Stripe Bypass**: Can create labels without immediate payment (pending payment records are created)

### Authentication Headers

All API endpoints support authentication using Bearer tokens:

```
Authorization: Bearer <your_token>
```

The token contains user identification and permissions. Some endpoints support optional authentication to retrieve basic information without authentication.

## Testing Mode

The API supports a testing mode for label creation without processing actual payments.

To use testing mode:

1. Use the QA environment URL instead of the production URL
2. Rate and duties calculations will still provide accurate pricing

QA Environment: `https://shippmate-server-test-976f52a5a04a.herokuapp.com`
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
      "capabilities": ["domestic", "international", "duties", "pickup"]
    },
    {
      "id": "bringer",
      "name": "Bringer",
      "capabilities": ["international", "duties"]
    }
  ]
}
```

### Get Shipping Rates

Calculates shipping rates from multiple carriers with a unified request format.

**Endpoint:** `POST /shipping/rate`
**Authentication:** Optional - authenticated users may receive custom rates based on their markup settings

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
    "code": "03", // Optional service code if you want calibrated rates for a specific service with provided dimensions and weight
    "name": "UPS Ground"
  },
  "shipmentDate": "2023-06-15",
  "declaredValue": 150, // Optional, for international shipments
  "currency": "USD",
  "incoterms": "DDP", // DDU is available only for export from US, DDP is mandatory for all imports to the US and for all Bringer shipments
  "documentsOnly": false
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

**Notes:**

- Rates are automatically sorted by transit time (fastest first), then by price
- For authenticated users, rates include user-specific markups based on their settings
- The API automatically selects the cheapest rate when multiple accounts are available for the same service

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
  "declaredValue": 170.0,
  "currency": "USD",
  "incoterms": "DDP", // DDU is available only for export from US, DDP is mandatory for all imports to the US and for all Bringer shipments
  "carrier": "ups", // Specify carrier to use for duties calculation (ups or bringer)
  "documentsOnly": false
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "duties": 15.3,
    "taxes": 8.5,
    "brokerageFees": 35.62,
    "total": 59.42,
    "currency": "USD",
    "domestic": false
  }
}
```

**Notes:**

- For Bringer carrier, duties are calculated individually for each item and then summed
- For UPS carrier, the API uses the UPS Landed Cost API for accurate calculations
- HS codes are validated and may be automatically adjusted for specific country combinations
- **DDP is mandatory for all imports to the US (regardless of carrier) and for all Bringer shipments.** If you do not provide incoterms, the system will enforce these rules automatically.

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
    "phone": "+13105551234",
    "taxId": "12345678901" // Required for Brazilian And Argentinian recipients when using Bringer carrier
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
  "documentsOnly": false,
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
  "declaredValue": 170.0,
  "currency": "USD",
  "incoterms": "DDP", // DDU is available only for export from US, DDP is mandatory for all imports to the US and for all Bringer shipments
  "shippingPurpose": "SALE",
  "pickup": {
    // Optional UPS pickup object
    "pickupDate": "2023-06-16",
    "pickupStart": "09:00",
    "pickupEnd": "17:00",
    "pickupType": "01",
    "residential": "Y",
    "pickupPoint": "Front Door",
    "floor": "3rd Floor",
    "specialInstructions": "Ring doorbell twice"
  }
}
```

**Important Notes:**

- For shipments to Brazil using Bringer carrier, a valid Tax ID (CPF) is required in the recipient information
- For Bringer shipments to Brazil (BR), Mexico (MX), Chile (CL), or Argentina (AR), states must be either valid ISO 2-letter codes or recognized state names. Use the `/shipping/limits` endpoint to get the list of accepted state formats
- For international shipments, providing HS codes for items is mandatory
- For UPS international shipments, declared value and shipping purpose are required
- The `owner` field is automatically set to the authenticated user's ID if not provided
- Users with `stripeBypass` flag don't need to provide payment information

**Label Creation Process:**

1. Validate input parameters
2. Verify carrier is available for the user
3. Calculate shipping rate
4. Calculate duties (if international shipment)
5. Calculate total cost (shipping + duties)
6. Process payment (unless user has stripeBypass)
7. Create shipping label with the carrier
8. Return label, tracking information, and cost breakdown

**Response for UPS:**

```json
{
  "success": true,
  "data": {
    "orderId": "order_id_12345",
    "trackingNumber": "1Z999AA10123456784",
    "labelUrl": "data:image/gif;base64,R0lGODlhAQABAI...", // Base64 encoded label image
    "barcode": "data:image/png;base64,iVBORw0KG...", // Base64 encoded barcode image
    "pickupRequestNumber": "PRN123456", // If pickup was requested
    "costs": {
      "shipping": 21.24,
      "duties": 59.42,
      "pickup": 5.0,
      "total": 85.66,
      "currency": "USD"
    }
  },
  "message": "Label created successfully with UPS"
}
```

**Response for Bringer (Multi-leg):**

```json
{
  "success": true,
  "data": {
    "firstLeg": {
      "orderId": "order_id_12345",
      "trackingNumber": "1Z999AA10123456784",
      "labelUrl": "data:image/gif;base64,R0lGODlhAQABAI...",
      "barcode": "data:image/png;base64,iVBORw0KG...",
      "carrier": "ups",
      "service": "UPS Ground"
    },
    "secondLeg": {
      "orderId": "order_id_67890",
      "trackingNumber": "BPS123456789",
      "labelUrl": "data:image/gif;base64,R0lGODlhAQABAI...",
      "barcode": "data:image/png;base64,iVBORw0KG...",
      "carrier": "bringer",
      "service": "Standard"
    },
    "costs": {
      "shipping": 29.99,
      "duties": 59.42,
      "pickup": 0,
      "total": 89.41,
      "currency": "USD"
    }
  },
  "message": "Multi-leg label created successfully with Bringer"
}
```

### Incoterms Handling for UPS Labels

When creating a shipping label with UPS, the system will automatically determine the appropriate incoterms if not explicitly provided in your request. The logic is as follows:

- **For UPS imports to the US (destination country is US):**
  - The incoterms will always be set to **DDP** (Delivered Duty Paid), regardless of the origin country.
- **For shipments with UPS where the origin country is the US and the destination country is NOT the US:**
  - The default incoterms will be set to **DDU** (Delivered Duty Unpaid).
- **For all other UPS shipments (including non-US origins and non-US destinations):**
  - The default incoterms will be set to **DDP** (Delivered Duty Paid).

This logic is implemented in the backend. If you do not specify the `incoterms` field in your label creation request, the system will apply this logic automatically. You can override this by explicitly providing `incoterms` in your request body.

In this case, the system will set `incoterms` to `DDP` for the UPS label (import to the US).

### Document-only Shipments

Some shipments consist exclusively of paper documents (e.g., contracts, certificates).  
Set **`"documentsOnly": true`** in your request to indicate this. The behaviour is:

- Only UPS carrier is supported currently for document labels.
- Duties and taxes are automatically returned as **0**.

Example excerpt:

```json
{
  "from": {
    /* … */
  },
  "to": {
    /* … */
  },
  "packages": [
    {
      "weight": "1",
      "weightUnit": "lb",
      "length": "12",
      "width": "9",
      "height": "1",
      "measurementUnit": "in"
    }
  ],
  "carrier": "ups",
  "service": { "code": "02", "name": "UPS 2nd Day Air" },
  "documentsOnly": true
}
```

If `documentsOnly` is omitted or set to **false** (default) the request is treated as a merchandise shipment.

### Track Shipment

Tracks a shipment with a unified interface supporting multiple carriers.

**Endpoint:** `POST /shipping/tracking`
**Authentication:** Optional - works the same for all users

**Request Body:**

```json
{
  "trackingNumber": "1Z999AA10123456784",
  "carrier": "ups",
  "originCountry": "US" // Required for UPS tracking
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

### Get Shipping Limits

Retrieves shipping limits for a specific carrier and destination, including accepted state formats for countries that require state validation.

**Endpoint:** `GET /shipping/limits`
**Authentication:** Optional - works the same for all users

**Request Parameters:**

```
carrier: string (required) - Carrier code (e.g., "bringer", "ups")
origin: string (required) - Origin country code (e.g., "US")
destination: string (required) - Destination country code (e.g., "AR", "BR", "CL", "MX")
dimensionUnit: string (optional) - "IN" or "CM" (default: "IN")
weightUnit: string (optional) - "LBS" or "KGS" (default: "LBS")
```

**Response:**

```json
{
  "success": true,
  "data": {
    "carrier": "bringer",
    "origin": "US",
    "destination": "AR",
    "dimensions": {
      "minLength": 6,
      "minWidth": 1,
      "minHeight": 4,
      "minSum": 11,
      "maxLength": 39,
      "maxWidth": 39,
      "maxHeight": 39,
      "maxSum": 70
    },
    "weight": {
      "max": 44
    },
    "states": {
      "acceptedNames": [
        { "name": "buenos aires", "isoCode": "BA" },
        { "name": "ciudad autonoma de buenos aires", "isoCode": "CABA" },
        { "name": "catamarca", "isoCode": "CT" },
        { "name": "chaco", "isoCode": "CH" },
        { "name": "cordoba", "isoCode": "CB" }
        // ... and more
      ],
      "isoCodes": ["BA", "CABA", "CT", "CH", "CB", "CR", "ER", "FM", "JY", "LP", "LR", "MZ", "MN", "NQ", "RN", "SA", "SJ", "SL", "SC", "SF", "SE", "TF", "TM"],
      "description": "Accepted state names can be either ISO 2-letter codes or full state names as shown in acceptedNames"
    }
  }
}
```

**Notes:**

- The `states` field is only included for Bringer shipments to countries that require state validation (AR, BR, CL, MX)
- State names are case-insensitive and accents are normalized during validation
- Both ISO 2-letter codes and full state names are accepted

### Validate Package Dimensions

Validates package dimensions, weight, and value against shipping limits.

**Endpoint:** `POST /shipping/validate-dimensions`
**Authentication:** Optional - works the same for all users

**Request Body:**

```json
{
  "carrier": "bringer",
  "dimensions": {
    "length": 35,
    "width": 25,
    "height": 15
  },
  "weight": 30,
  "value": 350,
  "currency": "USD",
  "origin": "US",
  "destination": "AR",
  "dimensionUnit": "IN",
  "weightUnit": "LBS"
}
```

**Response for Valid Package:**

```json
{
  "success": true,
  "data": {
    "carrier": "bringer",
    "valid": true,
    "dimensions": {
      "valid": true,
      "message": null
    },
    "weight": {
      "valid": true,
      "message": null
    },
    "value": {
      "valid": true,
      "message": null
    }
  }
}
```

**Response for Invalid Package:**

```json
{
  "success": true,
  "data": {
    "carrier": "bringer",
    "valid": false,
    "dimensions": {
      "valid": false,
      "message": "For AR via Bringer, maximum allowed length is 39 IN"
    },
    "weight": {
      "valid": true,
      "message": null
    },
    "value": {
      "valid": false,
      "message": "Shipments over US$400.00 require a quote with H.S. Code, full description, invoice price, weight and dimensions. Please contact bps-sales@bringer.com"
    },
    "message": "For AR via Bringer, maximum allowed length is 39 IN"
  }
}
```

### Create Pickup Request

Creates a pickup request for UPS shipments.

**Endpoint:** `POST /shipping/pickup`
**Authentication:** Required

**Request Body (with Order IDs):**

```json
{
  "orderIds": ["order_id_1", "order_id_2"],
  "pickup": {
    "pickupDate": "2023-06-16",
    "pickupStart": "09:00",
    "pickupEnd": "17:00",
    "pickupType": "01",
    "residential": "Y",
    "pickupPoint": "Front Door",
    "floor": "3rd Floor",
    "specialInstructions": "Please use service entrance"
  },
  "confirmationEmail": "pickup@example.com"
}
```

**Request Body (with Custom Request):**

```json
{
  "customRequest": {
    "from": {
      "name": "John Doe",
      "companyName": "Company Inc",
      "addressLine": "123 Main St",
      "city": "New York",
      "state": "NY",
      "postalCode": "10001",
      "country": "US",
      "phone": "+12125551234"
    },
    "to": {
      "name": "Jane Smith",
      "addressLine": "456 Other Ave",
      "city": "Los Angeles",
      "state": "CA",
      "postalCode": "90001",
      "country": "US"
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
    "service": {
      "code": "03",
      "name": "UPS Ground"
    }
  },
  "pickup": {
    "pickupDate": "2023-06-16",
    "pickupStart": "09:00",
    "pickupEnd": "17:00",
    "pickupType": "01",
    "residential": "Y",
    "pickupPoint": "Front Door",
    "floor": "3rd Floor",
    "specialInstructions": "Please use service entrance"
  }
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "pickupRequestNumber": "PRN123456",
    "pickupCharge": "5.00",
    "rateStatus": "VALID",
    "orderIds": ["order_id_1", "order_id_2"]
  },
  "message": "Pickup request created successfully"
}
```

**Pickup Parameters:**

- `pickupDate` (required): Date for pickup in YYYY-MM-DD format
- `pickupStart` (required): Earliest pickup time in HH:MM format (24-hour)
- `pickupEnd` (required): Latest pickup time in HH:MM format (24-hour)
- `pickupType` (required): Type of pickup service
  - `"01"`: Same-Day Pickup
  - `"02"`: Next-Day Pickup
  - `"03"`: A Specific-Day Pickup
- `residential` (required): Indicates if pickup location is residential
  - `"Y"`: Residential address
  - `"N"`: Commercial address
- `pickupPoint` (optional): Specific pickup location (e.g., "Front Door", "Reception", "Loading Dock")
- `floor` (optional): Floor or suite information (e.g., "3rd Floor", "Suite 200")
- `specialInstructions` (optional): Special instructions for the driver (max 255 characters)

**Notes:**

- Pickup is only supported for UPS shipments
- When using orderIds, all orders must have the same pickup address
- The pickup charge is automatically calculated and returned in the response
- For bulk orders, each package is counted separately in the pickup request
- For regular orders, item quantities are preserved in the pickup request

## Pickup Feature Best Practices

When using the pickup functionality through the API:

1. **Floor Information**: Be specific about the location within the building
   - Good: "3rd Floor, Suite 305", "Ground Floor - Reception"
   - Avoid: Just "3" or "Third"

2. **Special Instructions**: Keep instructions clear and concise
   - Include relevant contact information if needed
   - Mention any access codes or special entry procedures
   - Specify preferred entrances or loading areas
   - Maximum 255 characters

3. **Pickup Point Selection**: Choose the most appropriate pickup location
   - Available options include: "Front Door", "Reception", "Loading Dock", etc.
   - This helps drivers locate the exact pickup point

4. **Residential vs Commercial**: Correctly identify the pickup location type
   - Use `"Y"` for residential addresses
   - Use `"N"` for commercial addresses
   - This affects pickup scheduling and pricing

## Shipping Limits

### Bringer Limits by Destination

For shipments from the US using Bringer carrier, the following limits apply by destination country:

#### Argentina (AR)

- **Weight Limits:**
  - Maximum: 44 lbs (20 kg)
- **Dimension Limits:**
  - **Inches:** Length ≤ 39", Width ≤ 39", Height ≤ 39", Combined (L+W+H) ≤ 70"
  - **Centimeters:** Length ≤ 100 cm, Width ≤ 100 cm, Height ≤ 100 cm, Combined ≤ 180 cm
  - **Minimums:** Length ≥ 6" (16 cm), Width ≥ 1" (1 cm), Height ≥ 4" (11 cm)
- **Value Limits:**
  - Maximum declared value: US$400
  - Shipments over US$400 require a quote with H.S. Code, full description, invoice price, weight and dimensions
  - Contact: bps-sales@bringer.com

#### Brazil (BR)

- **Weight Limits:**
  - Maximum: 66 lbs (29.9 kg)
- **Dimension Limits:**
  - **Inches:** No individual dimension limits, Combined (L+W+H) ≤ 70"
  - **Centimeters:** No individual dimension limits, Combined ≤ 180 cm
  - **Minimums:** Length ≥ 6" (16 cm), Width ≥ 1" (1 cm), Height ≥ 4" (11 cm)

#### Chile (CL)

- **Weight Limits:**
  - Maximum: 6 lbs (2.7 kg)
- **Dimension Limits:**
  - **Inches:** Length ≤ 39.85", Width ≤ 39.85", Height ≤ 39.85", Combined ≤ 118"
  - **Centimeters:** Length ≤ 100 cm, Width ≤ 100 cm, Height ≤ 100 cm, Combined ≤ 300 cm
  - **Minimums:** All dimensions ≥ 1" (1 cm)

#### Mexico (MX)

- **Weight Limits:**
  - Maximum: 132 lbs (59.8 kg)
- **Dimension Limits:**
  - **Inches:** Length ≤ 23", No width/height limits, Combined (L+W+H) ≤ 70"
  - **Centimeters:** Length ≤ 60 cm, No width/height limits, Combined ≤ 180 cm
  - **Minimums:** Length ≥ 6" (16 cm), Width ≥ 1" (1 cm), Height ≥ 4" (11 cm)

### UPS Limits

UPS has standard dimensional limits that apply globally:

- **Maximum Length:** 108 inches (274 cm)
- **Maximum Girth:** Length + 2×(Width + Height) ≤ 165 inches (419 cm)
- **Weight Limits:** Vary by service and destination

### Value Limits

#### Argentina

- **Maximum Value:** US$400
- **Currency:** USD only
- **Over-limit Requirements:** Shipments exceeding US$400 require:
  - Complete H.S. Code classification
  - Full product description
  - Invoice price documentation
  - Weight and dimension specifications
  - Quote request to bps-sales@bringer.com

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
- `401`: Unauthorized - Authentication required or invalid token
- `404`: Not Found - Resource not found
- `500`: Internal Server Error - Server-side error

For validation errors, the response will include specific information about the missing or invalid fields.

## Country-Specific Requirements

### State Validation for Bringer Shipments

When using Bringer carrier for shipments to Brazil (BR), Mexico (MX), Chile (CL), or Argentina (AR), state validation is enforced:

- **Accepted Formats:**
  - ISO 2-letter state codes (e.g., "SP" for São Paulo, "BC" for Baja California)
  - Full state names in lowercase (e.g., "são paulo", "baja california")
  - State names are case-insensitive and accents are normalized

- **Validation Process:**
  - If the state is already a 2-letter code, it's validated against known ISO codes for that country
  - If the state is a full name, it's converted to its ISO code automatically
  - If the state is neither a valid ISO code nor a recognized state name, the API returns an error

- **Error Messages:** When state validation fails, you'll receive a descriptive error like:
  ```
  Invalid origin state: State 'XY' is not a valid 2-letter ISO code and was not found in our list of accepted state names for BR
  ```

- **Getting Valid States:** Use the `/shipping/limits` endpoint to retrieve the list of accepted state names and ISO codes for each country

### Argentina (AR)

When using Bringer carrier for shipments to Argentina:

- **State Validation:** Required - see State Validation section above
- **Value Limitation:** Maximum declared value is US$400
- **Over-limit Process:** Shipments exceeding US$400 require a comprehensive quote including:
  - Complete H.S. Code classification for all items
  - Detailed product descriptions
  - Invoice pricing documentation
  - Exact weight and dimension specifications
  - Direct contact with bps-sales@bringer.com for approval
- **Weight and Dimension Limits:** Must comply with Bringer Argentina limits (see Shipping Limits section)

### Brazil (BR)

When using Bringer carrier for shipments to Brazil:

- **State Validation:** Required - see State Validation section above
- A valid Tax ID (CPF for individuals, CNPJ for businesses) is required in the recipient information
- Must be provided in the `taxId` field of the `to` address object
- The API will return a validation error if the Tax ID is missing

### Chile (CL)

When using Bringer carrier for shipments to Chile:

- **State Validation:** Required - see State Validation section above
- Standard weight and dimension limits apply (see Shipping Limits section)

### Mexico (MX)

When using Bringer carrier for shipments to Mexico:

- **State Validation:** Required - see State Validation section above
- Shipments are automatically routed through Laredo, TX for optimal pricing
- Standard weight and dimension limits apply (see Shipping Limits section)

### International Shipments

For all international shipments:

- HS codes are required for all items
- HS codes must be valid for both origin and destination countries
- Declared value is required for customs clearance
- Shipping purpose must be specified (SALE, GIFT, SAMPLE, RETURN, OTHER)

