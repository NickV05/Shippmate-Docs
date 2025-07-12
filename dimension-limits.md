# Shipping Dimension and Weight Limits

This document provides detailed information about shipping limits for different carriers and destinations.

## Table of Contents

- [Overview](#overview)
- [Bringer Carrier Limits](#bringer-carrier-limits)
  - [Argentina (AR)](#argentina-ar)
  - [Brazil (BR)](#brazil-br)
  - [Chile (CL)](#chile-cl)
  - [Mexico (MX)](#mexico-mx)
- [UPS Carrier Limits](#ups-carrier-limits)
- [Value Restrictions](#value-restrictions)
- [API Usage](#api-usage)
- [Validation Examples](#validation-examples)

## Overview

Different carriers have specific limits for package dimensions, weight, and declared value depending on the destination country. These limits are enforced by the Shipping API to ensure compliance with carrier requirements.

## Bringer Carrier Limits

For shipments from the US using Bringer carrier, the following limits apply:

### State Validation Requirements

**Important:** All Bringer shipments to Brazil, Mexico, Chile, and Argentina require valid state/province information. The API validates states and accepts both:
- **ISO Codes:** BR (São Paulo), RJ (Rio de Janeiro), etc.
- **Full Names:** São Paulo, Rio de Janeiro, etc.

**Validation Process:**
- Case-insensitive matching with accent normalization
- Automatic conversion from full names to ISO codes
- Descriptive error messages for invalid states
- Use `/shipping/limits` endpoint to retrieve valid state formats

**Error Example:**
```json
{
  "error": "Invalid state 'SP1' for Brazil. Valid states include: AC, AL, AP, AM, BA, CE, DF, ES, GO, MA, MT, MS, MG, PA, PB, PR, PE, PI, RJ, RN, RS, RO, RR, SC, SP, SE, TO. You can also use full state names like 'São Paulo'."
}
```

### Argentina (AR)

#### Weight Limits

- **Maximum Weight:** 44 lbs (20 kg)

#### Dimension Limits

- **Imperial (Inches):**
  - Maximum Length: 39"
  - Maximum Width: 39"
  - Maximum Height: 39"
  - Maximum Combined (L+W+H): 70"
  - Minimum Length: 6"
  - Minimum Width: 1"
  - Minimum Height: 4"

- **Metric (Centimeters):**
  - Maximum Length: 100 cm
  - Maximum Width: 100 cm
  - Maximum Height: 100 cm
  - Maximum Combined (L+W+H): 180 cm
  - Minimum Length: 16 cm
  - Minimum Width: 1 cm
  - Minimum Height: 11 cm

#### Value Limits

- **Maximum Declared Value:** US$400
- **Currency:** USD only
- **Over-limit Process:** Shipments exceeding US$400 require a comprehensive quote including:
  - Complete H.S. Code classification for all items
  - Detailed product descriptions
  - Invoice pricing documentation
  - Exact weight and dimension specifications
  - Direct contact with bps-sales@bringer.com for approval

### Brazil (BR)

#### Weight Limits

- **Maximum Weight:** 66 lbs (29.9 kg)

#### Dimension Limits

- **Imperial (Inches):**
  - Maximum Length: No limit
  - Maximum Width: No limit
  - Maximum Height: No limit
  - Maximum Combined (L+W+H): 70"
  - Minimum Length: 6"
  - Minimum Width: 1"
  - Minimum Height: 4"

- **Metric (Centimeters):**
  - Maximum Length: No limit
  - Maximum Width: No limit
  - Maximum Height: No limit
  - Maximum Combined (L+W+H): 180 cm
  - Minimum Length: 16 cm
  - Minimum Width: 1 cm
  - Minimum Height: 11 cm

### Chile (CL)

#### Weight Limits

- **Maximum Weight:** 6 lbs (2.7 kg)

#### Dimension Limits

- **Imperial (Inches):**
  - Maximum Length: 39.85"
  - Maximum Width: 39.85"
  - Maximum Height: 39.85"
  - Maximum Combined (L+W+H): 118"
  - Minimum Length: 1"
  - Minimum Width: 1"
  - Minimum Height: 1"

- **Metric (Centimeters):**
  - Maximum Length: 100 cm
  - Maximum Width: 100 cm
  - Maximum Height: 100 cm
  - Maximum Combined (L+W+H): 300 cm
  - Minimum Length: 1 cm
  - Minimum Width: 1 cm
  - Minimum Height: 1 cm

### Mexico (MX)

#### Weight Limits

- **Maximum Weight:** 132 lbs (59.8 kg)

#### Dimension Limits

- **Imperial (Inches):**
  - Maximum Length: 23"
  - Maximum Width: No limit
  - Maximum Height: No limit
  - Maximum Combined (L+W+H): 70"
  - Minimum Length: 6"
  - Minimum Width: 1"
  - Minimum Height: 4"

- **Metric (Centimeters):**
  - Maximum Length: 60 cm
  - Maximum Width: No limit
  - Maximum Height: No limit
  - Maximum Combined (L+W+H): 180 cm
  - Minimum Length: 16 cm
  - Minimum Width: 1 cm
  - Minimum Height: 11 cm

## UPS Carrier Limits

UPS has standard dimensional limits that apply globally:

- **Maximum Length:** 108 inches (274 cm)
- **Maximum Girth:** Length + 2×(Width + Height) ≤ 165 inches (419 cm)
- **Weight Limits:** Vary by service and destination (generally up to 150 lbs/70 kg for most services)

## Value Restrictions

### Argentina

- **Maximum Value:** US$400
- **Currency:** USD only
- **Enforcement:** API will reject shipments exceeding this limit
- **Over-limit Requirements:**
  - Quote request to bps-sales@bringer.com
  - Complete documentation including H.S. codes
  - Full product descriptions and invoice pricing

## API Usage

### Get Limits for a Destination

```bash
GET /shipping/limits?carrier=bringer&origin=US&destination=AR&dimensionUnit=IN&weightUnit=LBS
```

**Enhanced Response with State Information:**

```json
{
  "success": true,
  "data": {
    "dimensions": {
      "maxLength": 39,
      "maxWidth": 39,
      "maxHeight": 39,
      "maxCombined": 70,
      "minLength": 6,
      "minWidth": 1,
      "minHeight": 4
    },
    "weight": {
      "max": 44,
      "unit": "LBS"
    },
    "value": {
      "max": 400,
      "currency": "USD"
    },
    "states": {
      "required": true,
      "acceptedNames": [
        "Buenos Aires", "Catamarca", "Chaco", "Chubut", "Córdoba", 
        "Corrientes", "Entre Ríos", "Formosa", "Jujuy", "La Pampa", 
        "La Rioja", "Mendoza", "Misiones", "Neuquén", "Río Negro", 
        "Salta", "San Juan", "San Luis", "Santa Cruz", "Santa Fe", 
        "Santiago del Estero", "Tierra del Fuego", "Tucumán", "Ciudad Autónoma de Buenos Aires"
      ],
      "isoCodes": [
        "AR-B", "AR-K", "AR-H", "AR-U", "AR-X", "AR-W", "AR-E", 
        "AR-P", "AR-Y", "AR-L", "AR-F", "AR-M", "AR-N", "AR-Q", 
        "AR-R", "AR-A", "AR-J", "AR-D", "AR-Z", "AR-S", "AR-G", 
        "AR-V", "AR-T", "AR-C"
      ],
      "description": "Argentina requires valid province information. Use either full province names or ISO codes."
    }
  }
}
```

### Validate Package Dimensions

```bash
POST /shipping/validate-dimensions
Content-Type: application/json

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

## Validation Examples

### Valid Package to Argentina

**Request:**

```json
{
  "carrier": "bringer",
  "dimensions": { "length": 35, "width": 25, "height": 8 },
  "weight": 35,
  "value": 350,
  "origin": "US",
  "destination": "AR",
  "dimensionUnit": "IN",
  "weightUnit": "LBS"
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "valid": true,
    "dimensions": { "valid": true, "message": null },
    "weight": { "valid": true, "message": null },
    "value": { "valid": true, "message": null }
  }
}
```

### Oversized Package

**Request:**

```json
{
  "carrier": "bringer",
  "dimensions": { "length": 45, "width": 25, "height": 8 },
  "weight": 35,
  "origin": "US",
  "destination": "AR"
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "valid": false,
    "dimensions": {
      "valid": false,
      "message": "For AR via Bringer, maximum allowed length is 39 IN"
    },
    "weight": { "valid": true, "message": null },
    "value": { "valid": true, "message": null }
  }
}
```

### Over-Value Package to Argentina

**Request:**

```json
{
  "carrier": "bringer",
  "dimensions": { "length": 35, "width": 25, "height": 8 },
  "weight": 35,
  "value": 500,
  "origin": "US",
  "destination": "AR"
}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "valid": false,
    "dimensions": { "valid": true, "message": null },
    "weight": { "valid": true, "message": null },
    "value": {
      "valid": false,
      "message": "Shipments over US$400.00 require a quote with H.S. Code, full description, invoice price, weight and dimensions. Please contact bps-sales@bringer.com"
    }
  }
}
```

## Notes

1. **Combined Dimensions:** Some countries have restrictions on the combined dimensions (L+W+H) in addition to individual dimension limits.

2. **Minimum Dimensions:** All countries have minimum dimension requirements to ensure packages can be properly handled.

3. **Weight vs. Dimension Priority:** Both weight and dimension limits must be met. A package failing either check will be rejected.

4. **Value Validation:** Currently only Argentina has specific value restrictions, but this may be expanded to other countries in the future.

5. **Currency Support:** Value limits currently only support USD. Other currencies may be added in future updates.
