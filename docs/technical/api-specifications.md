# API Specifications for Elite Locadora

## 🌐 RESTful API Overview

### API Design Principles
- **RESTful Architecture** - Standard HTTP methods and status codes
- **JSON Data Format** - Consistent request/response structure
- **Versioning Strategy** - URL path versioning (/api/v1/)
- **Authentication** - JWT-based token authentication
- **Rate Limiting** - Prevent abuse with configurable limits
- **Comprehensive Documentation** - OpenAPI 3.0 specification

### Base URL Structure
```
Production: https://api.elitelocadora.com/v1
Staging: https://staging-api.elitelocadora.com/v1
Development: http://localhost:8000/api/v1
```

## 🔐 Authentication Endpoints

### Authentication Flow
**POST /api/v1/auth/login**
```json
{
  "email": "customer@example.com",
  "password": "secure_password",
  "remember_me": true
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 123,
      "name": "John Doe",
      "email": "customer@example.com",
      "role": "customer",
      "verified_at": "2024-01-15T10:30:00Z"
    },
    "tokens": {
      "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
      "refresh_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9...",
      "expires_in": 3600,
      "token_type": "Bearer"
    }
  },
  "message": "Authentication successful"
}
```

### User Registration
**POST /api/v1/auth/register**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secure_password",
  "password_confirmation": "secure_password",
  "phone": "+1234567890",
  "date_of_birth": "1990-05-15",
  "license_number": "D1234567890",
  "license_expiry": "2026-05-15"
}
```

### Token Management
**POST /api/v1/auth/refresh** - Refresh access token
**POST /api/v1/auth/logout** - Invalidate current session
**POST /api/v1/auth/logout-all** - Invalidate all user sessions

## 🚗 Vehicle Management API

### Vehicle Search
**GET /api/v1/vehicles/search**

**Query Parameters:**
```
pickup_location: string (required)
return_location: string (optional, defaults to pickup_location)
pickup_date: datetime (required, ISO 8601)
return_date: datetime (required, ISO 8601)
category: string (optional, filter by vehicle category)
min_price: number (optional, minimum daily rate)
max_price: number (optional, maximum daily rate)
features: array (optional, required features)
sort: string (optional, price|rating|popularity)
page: number (optional, default 1)
per_page: number (optional, default 20, max 100)
```

**Response:**
```json
{
  "success": true,
  "data": {
    "vehicles": [
      {
        "id": 101,
        "make": "Toyota",
        "model": "Camry",
        "year": 2023,
        "category": {
          "id": 3,
          "name": "Mid-size",
          "slug": "mid-size"
        },
        "features": [
          "automatic_transmission",
          "bluetooth",
          "gps_navigation",
          "backup_camera"
        ],
        "specifications": {
          "seats": 5,
          "doors": 4,
          "transmission": "automatic",
          "fuel_type": "gasoline",
          "mpg_city": 28,
          "mpg_highway": 39
        },
        "pricing": {
          "base_rate": 65.00,
          "total_cost": 195.00,
          "currency": "USD",
          "includes_taxes": false,
          "breakdown": {
            "daily_rate": 65.00,
            "days": 3,
            "taxes": 19.50,
            "fees": 15.00
          }
        },
        "availability": {
          "status": "available",
          "pickup_location": {
            "id": 1,
            "name": "Downtown Location",
            "address": "123 Main St, City, State 12345"
          },
          "return_location": {
            "id": 1,
            "name": "Downtown Location", 
            "address": "123 Main St, City, State 12345"
          }
        },
        "images": [
          {
            "url": "https://cdn.elitelocadora.com/vehicles/101/main.jpg",
            "alt": "Toyota Camry - Main View",
            "is_primary": true
          }
        ],
        "rating": {
          "average": 4.6,
          "count": 127,
          "distribution": {
            "5": 89,
            "4": 25,
            "3": 8,
            "2": 3,
            "1": 2
          }
        }
      }
    ],
    "pagination": {
      "current_page": 1,
      "last_page": 5,
      "per_page": 20,
      "total": 97,
      "from": 1,
      "to": 20
    },
    "filters": {
      "applied": {
        "category": "mid-size",
        "max_price": 100
      },
      "available": {
        "categories": [...],
        "price_range": {
          "min": 25.00,
          "max": 250.00
        },
        "features": [...]
      }
    }
  }
}
```

### Vehicle Details
**GET /api/v1/vehicles/{id}**

**Response includes:**
- Complete vehicle specifications
- High-resolution image gallery
- Customer reviews and ratings
- Availability calendar
- Pricing for different date ranges
- Location-specific information
- Terms and conditions
- Insurance options

## 📅 Booking Management API

### Create Booking Quote
**POST /api/v1/bookings/quote**
```json
{
  "vehicle_id": 101,
  "pickup_date": "2024-02-15T10:00:00Z",
  "return_date": "2024-02-18T10:00:00Z",
  "pickup_location_id": 1,
  "return_location_id": 1,
  "driver_age": 32,
  "add_ons": [
    {
      "type": "gps_navigation",
      "quantity": 1
    },
    {
      "type": "child_seat",
      "subtype": "infant",
      "quantity": 1
    }
  ],
  "insurance_options": [
    "collision_damage_waiver",
    "liability_insurance"
  ]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "quote_id": "quote_abc123",
    "expires_at": "2024-02-10T15:30:00Z",
    "pricing": {
      "subtotal": 195.00,
      "taxes": 19.50,
      "fees": 15.00,
      "add_ons": 45.00,
      "insurance": 30.00,
      "total": 304.50,
      "currency": "USD",
      "breakdown": {
        "vehicle_rental": {
          "daily_rate": 65.00,
          "days": 3,
          "subtotal": 195.00
        },
        "add_ons": [
          {
            "name": "GPS Navigation",
            "daily_rate": 10.00,
            "days": 3,
            "subtotal": 30.00
          },
          {
            "name": "Infant Car Seat",
            "daily_rate": 5.00,
            "days": 3,
            "subtotal": 15.00
          }
        ],
        "taxes_and_fees": [
          {
            "name": "State Tax",
            "rate": 0.10,
            "amount": 19.50
          },
          {
            "name": "Airport Fee",
            "amount": 15.00
          }
        ]
      }
    },
    "terms": {
      "cancellation_policy": "free_cancellation_48h",
      "modification_policy": "free_modification_24h",
      "mileage_allowance": "unlimited",
      "fuel_policy": "full_to_full"
    }
  }
}
```

### Create Booking
**POST /api/v1/bookings**
```json
{
  "quote_id": "quote_abc123",
  "primary_driver": {
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "+1234567890",
    "date_of_birth": "1990-05-15",
    "license": {
      "number": "D1234567890",
      "expiry_date": "2026-05-15",
      "issuing_state": "CA",
      "issuing_country": "US"
    },
    "address": {
      "street": "123 Main St",
      "city": "San Francisco",
      "state": "CA",
      "postal_code": "94102",
      "country": "US"
    }
  },
  "additional_drivers": [
    {
      "name": "Jane Doe",
      "date_of_birth": "1992-08-20",
      "license": {
        "number": "D0987654321",
        "expiry_date": "2025-08-20",
        "issuing_state": "CA",
        "issuing_country": "US"
      }
    }
  ],
  "payment": {
    "method": "credit_card",
    "card": {
      "token": "card_token_xyz789",
      "billing_address": {
        "street": "123 Main St",
        "city": "San Francisco",
        "state": "CA",
        "postal_code": "94102",
        "country": "US"
      }
    }
  },
  "special_requests": [
    "Please have vehicle ready 15 minutes before pickup time",
    "Preferred parking near entrance"
  ]
}
```

### Booking Management
**GET /api/v1/bookings** - List user bookings
**GET /api/v1/bookings/{id}** - Get booking details
**PUT /api/v1/bookings/{id}** - Modify booking
**DELETE /api/v1/bookings/{id}** - Cancel booking

## 💳 Payment Processing API

### Payment Intent Creation
**POST /api/v1/payments/intent**
```json
{
  "booking_id": 12345,
  "amount": 304.50,
  "currency": "USD",
  "payment_method": "credit_card",
  "capture_method": "automatic",
  "metadata": {
    "booking_reference": "EL-2024-001234",
    "customer_id": 123
  }
}
```

### Payment Confirmation
**POST /api/v1/payments/confirm**
```json
{
  "payment_intent_id": "pi_abc123",
  "payment_method": "pm_xyz789"
}
```

### Refund Processing
**POST /api/v1/payments/refund**
```json
{
  "payment_id": "pay_abc123",
  "amount": 150.00,
  "reason": "cancellation",
  "metadata": {
    "cancellation_reason": "customer_request",
    "processed_by": "admin_user_456"
  }
}
```

## 📍 Location Services API

### Location Search
**GET /api/v1/locations/search**
```
query: string (search term)
latitude: number (for distance calculation)
longitude: number (for distance calculation)
radius: number (search radius in miles/km)
type: string (pickup|return|both)
```

### Location Details
**GET /api/v1/locations/{id}**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Downtown Location",
    "code": "DTN",
    "type": "full_service",
    "address": {
      "street": "123 Main Street",
      "city": "San Francisco",
      "state": "CA",
      "postal_code": "94102",
      "country": "US",
      "coordinates": {
        "latitude": 37.7749,
        "longitude": -122.4194
      }
    },
    "contact": {
      "phone": "+1-555-0123",
      "email": "downtown@elitelocadora.com"
    },
    "hours": {
      "monday": {"open": "08:00", "close": "18:00"},
      "tuesday": {"open": "08:00", "close": "18:00"},
      "wednesday": {"open": "08:00", "close": "18:00"},
      "thursday": {"open": "08:00", "close": "18:00"},
      "friday": {"open": "08:00", "close": "18:00"},
      "saturday": {"open": "09:00", "close": "17:00"},
      "sunday": {"open": "10:00", "close": "16:00"}
    },
    "services": [
      "pickup",
      "return",
      "after_hours_dropoff",
      "vehicle_inspection",
      "fuel_service"
    ],
    "amenities": [
      "parking_available",
      "wheelchair_accessible",
      "shuttle_service",
      "customer_lounge"
    ],
    "fleet_count": 45,
    "average_rating": 4.7
  }
}
```

## 📊 Analytics & Reporting API

### Admin Analytics Dashboard
**GET /api/v1/admin/analytics/dashboard**
```json
{
  "success": true,
  "data": {
    "period": {
      "start_date": "2024-01-01",
      "end_date": "2024-01-31"
    },
    "metrics": {
      "total_revenue": 125000.00,
      "total_bookings": 450,
      "average_booking_value": 277.78,
      "fleet_utilization": 0.72,
      "customer_satisfaction": 4.6,
      "cancellation_rate": 0.08
    },
    "comparisons": {
      "revenue_change": 0.15,
      "booking_change": 0.12,
      "utilization_change": 0.05
    },
    "charts": {
      "revenue_trend": [...],
      "booking_volume": [...],
      "popular_vehicles": [...],
      "location_performance": [...]
    }
  }
}
```

## 🔒 Error Handling & Status Codes

### Standard HTTP Status Codes
- **200 OK** - Successful request
- **201 Created** - Resource created successfully
- **400 Bad Request** - Invalid request parameters
- **401 Unauthorized** - Authentication required
- **403 Forbidden** - Insufficient permissions
- **404 Not Found** - Resource not found
- **422 Unprocessable Entity** - Validation errors
- **429 Too Many Requests** - Rate limit exceeded
- **500 Internal Server Error** - Server error

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The given data was invalid.",
    "details": {
      "email": ["The email field is required."],
      "pickup_date": ["The pickup date must be in the future."]
    },
    "trace_id": "req_abc123xyz"
  }
}
```

### Common Error Codes
- **INVALID_CREDENTIALS** - Authentication failed
- **VALIDATION_ERROR** - Request validation failed
- **RESOURCE_NOT_FOUND** - Requested resource not found
- **PERMISSION_DENIED** - Insufficient permissions
- **RATE_LIMIT_EXCEEDED** - Too many requests
- **VEHICLE_UNAVAILABLE** - Vehicle not available for dates
- **PAYMENT_FAILED** - Payment processing failed
- **BOOKING_CONFLICT** - Booking conflicts with existing reservation

## 🔐 Security Specifications

### Authentication & Authorization
- **JWT Tokens** - Secure token-based authentication
- **Role-based Access Control** - Customer, Staff, Admin, Super Admin
- **Token Expiration** - Short-lived access tokens with refresh capability
- **Rate Limiting** - Configurable limits per endpoint and user role
- **CORS Configuration** - Proper cross-origin resource sharing setup

### Data Protection
- **Input Validation** - Comprehensive request validation
- **SQL Injection Prevention** - Parameterized queries and ORM usage
- **XSS Prevention** - Output encoding and CSP headers
- **CSRF Protection** - Token-based CSRF protection
- **Data Encryption** - Sensitive data encryption at rest and in transit

### API Security Headers
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```