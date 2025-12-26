# GFinder API Documentation

Complete API reference for GFinder Backend.

## Base URL
```
http://localhost:3000/api
```

## Authentication

Most endpoints require authentication via JWT token. Include the token in the Authorization header:
```
Authorization: Bearer <your_jwt_token>
```

---

## 🔐 Authentication APIs (`/api/auth`)

### POST `/api/auth/login`
Login with email and password.

**Authentication:** Not required (public endpoint)

**Request Headers:**
```
Content-Type: application/json
```

**Request Body (Required):**
```json
{
  "email": "user@example.com",        // Required: string, valid email format
  "password": "password123",           // Required: string
  "rememberMe": true                   // Optional: boolean, defaults to false
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe",
    "role": "CUSTOMER",
    "mobileNumber": "+1234567890",
    "country": "USA",
    "city": "New York",
    "profileImage": "/uploads/images/profile.jpg",
    "rememberMe": true,
    "language": "ENGLISH",
    "customerProfile": { ... },
    // OR "serviceProviderProfile": { ... } if role is SERVICE_PROVIDER
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: Missing email or password
- `401 Unauthorized`: Invalid email or password

---

### POST `/api/auth/customer/signup`
Register a new customer account.

**Authentication:** Not required (public endpoint)

**Request Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "name": "John Doe",                  // Required: string
  "email": "customer@example.com",     // Required: string, valid email format, must be unique
  "mobileNumber": "+1234567890",       // Optional: string
  "country": "USA",                    // Optional: string
  "city": "New York"                   // Optional: string
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Customer account created successfully",
  "token": "jwt_token_here",
  "user": {
    "id": "uuid",
    "email": "customer@example.com",
    "name": "John Doe",
    "role": "CUSTOMER",
    "mobileNumber": "+1234567890",
    "country": "USA",
    "city": "New York",
    "customerProfile": {
      "id": "uuid",
      "userId": "uuid"
    },
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: Missing required fields (name, email) or email already exists

---

### POST `/api/auth/service-provider/signup`
Register a new service provider account (status: PENDING, requires admin approval).

**Authentication:** Not required (public endpoint)

**Request Headers:**
```
Content-Type: multipart/form-data
```

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `profileImage` | File | No | Profile image (JPEG, PNG, GIF, WebP) |
| `name` | String | **Yes** | Service provider name |
| `email` | String | **Yes** | Email address (must be unique) |
| `mobileNumber` | String | No | Mobile phone number |
| `whatsappNumber` | String | No | WhatsApp number with country code |
| `country` | String | No | Country name |
| `city` | String | No | City name |
| `description` | String | No | Service description |
| `serviceIds` | JSON String/Array | No | Array of service IDs: `["uuid1", "uuid2"]` |
| `subServiceIds` | JSON String/Array | No | Array of sub-service IDs: `["uuid1", "uuid2"]` |
| `availability` | JSON String/Array | No | Availability array (see format below) |

**Availability Array Format:**
```json
[
  {
    "dayOfWeek": 1,           // Required: integer (0=Sunday, 1=Monday, ..., 6=Saturday)
    "startTime": "09:00",     // Required: string, format "HH:MM"
    "endTime": "17:00",       // Required: string, format "HH:MM"
    "isAvailable": true       // Optional: boolean, defaults to true
  }
]
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Service provider account created successfully. Please wait for admin approval.",
  "token": "jwt_token_here",
  "user": {
    "id": "uuid",
    "email": "provider@example.com",
    "name": "Clean Pro",
    "role": "SERVICE_PROVIDER",
    "mobileNumber": "+1234567890",
    "whatsappNumber": "+1234567890",
    "country": "USA",
    "city": "New York",
    "profileImage": "/uploads/images/profile.jpg",
    "serviceProviderProfile": {
      "id": "uuid",
      "status": "PENDING",
      "description": "Professional cleaning services",
      "portfolioImages": [],
      "services": [ ... ],
      "availability": [ ... ]
    },
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: Missing required fields (name, email), email already exists, or invalid JSON format

---

### POST `/api/auth/forgot-password`
Request password reset OTP (sent via email). Unlimited OTP resends allowed.

**Authentication:** Not required (public endpoint)

**Request Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "user@example.com"  // Required: string, valid email format
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "If the email exists, an OTP has been sent"
}
```
*Note: Response message is generic for security (doesn't reveal if email exists)*

**Error Responses:**
- `400 Bad Request`: Missing email

---

### POST `/api/auth/resend-otp`
Resend password reset OTP. Unlimited resends allowed.

**Authentication:** Not required (public endpoint)

**Request Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "user@example.com"  // Required: string, valid email format
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "If the email exists, an OTP has been sent"
}
```

**Error Responses:**
- `400 Bad Request`: Missing email

---

### POST `/api/auth/reset-password`
Reset password using OTP code.

**Authentication:** Not required (public endpoint)

**Request Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "email": "user@example.com",        // Required: string, valid email format
  "otp": "123456",                    // Required: string, 6-digit OTP code
  "newPassword": "newPassword123",    // Required: string
  "confirmPassword": "newPassword123" // Required: string, must match newPassword
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password reset successfully"
}
```

**Error Responses:**
- `400 Bad Request`: Missing fields, passwords don't match, or invalid/expired OTP
- `404 Not Found`: User not found

---

### GET `/api/auth/me`
Get current authenticated user details.

**Authentication:** Required (Bearer token)

**Request Headers:**
```
Authorization: Bearer <your_jwt_token>
```

**Path Parameters:** None

**Query Parameters:** None

**Response (200 OK):**
```json
{
  "success": true,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe",
    "role": "CUSTOMER",
    "mobileNumber": "+1234567890",
    "country": "USA",
    "city": "New York",
    "profileImage": "/uploads/images/profile.jpg",
    "rememberMe": false,
    "language": "ENGLISH",
    "latitude": 40.7128,
    "longitude": -74.0060,
    "customerProfile": {
      "id": "uuid",
      "userId": "uuid"
    },
    // OR "serviceProviderProfile": { ... } if role is SERVICE_PROVIDER
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `401 Unauthorized`: Missing or invalid token, user not found

---

### POST `/api/auth/logout`
Logout (client should remove token from storage).

**Authentication:** Required (Bearer token)

**Request Headers:**
```
Authorization: Bearer <your_jwt_token>
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

## 👤 Customer APIs (`/api/customer`)

**All endpoints require:** Customer authentication (Bearer token)
**Base Path:** `/api/customer`

### PUT `/api/customer/location`
Update customer location (required for filtering service providers by radius).

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "latitude": 40.7128,   // Required: number, valid latitude (-90 to 90)
  "longitude": -74.0060  // Required: number, valid longitude (-180 to 180)
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Location updated successfully",
  "user": {
    "latitude": 40.7128,
    "longitude": -74.0060
  }
}
```

**Error Responses:**
- `400 Bad Request`: Missing latitude or longitude
- `401 Unauthorized`: Invalid or missing token

---

### PUT `/api/customer/language`
Update customer language preference.

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "language": "ENGLISH"  // Required: string, must be "ENGLISH" or "URDU"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Language updated successfully"
}
```

**Error Responses:**
- `400 Bad Request`: Invalid language value (must be "ENGLISH" or "URDU")

---

### GET `/api/customer/home`
Get home screen data (categories and service providers with location-based filtering).

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `categoryId` | String | No | Filter by category ID |
| `subServiceId` | String | No | Filter by sub-service ID |
| `radius` | Number | No | Filter radius in kilometers (default: 1) |
| `search` | String | No | Search query (searches name, category, title, sub-category) |

**Request Body:** None

**Example Request:**
```
GET /api/customer/home?radius=5&search=cleaning&categoryId=cat-123
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "categories": [
      {
        "id": "uuid",
        "title": "Cleaning",
        "icon": "/uploads/images/icon.png",
        "services": [
          {
            "id": "uuid",
            "title": "Home Cleaning",
            "icon": "/uploads/images/service-icon.png",
            "subServices": [
              {
                "id": "uuid",
                "title": "Deep Cleaning",
                "createdAt": "2024-01-01T00:00:00.000Z"
              }
            ]
          }
        ],
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "serviceProviders": [
      {
        "id": "uuid",
        "description": "Professional cleaning services",
        "portfolioImages": ["/uploads/images/img1.jpg"],
        "status": "ACTIVE",
        "user": {
          "id": "uuid",
          "name": "Clean Pro",
          "email": "clean@example.com",
          "mobileNumber": "+1234567890",
          "whatsappNumber": "+1234567890",
          "country": "USA",
          "city": "New York",
          "profileImage": "/uploads/images/profile.jpg",
          "latitude": 40.7128,
          "longitude": -74.0060
        },
        "services": [
          {
            "id": "uuid",
            "serviceId": "uuid",
            "service": {
              "id": "uuid",
              "title": "Home Cleaning",
              "category": {
                "id": "uuid",
                "title": "Cleaning"
              },
              "subServices": [ ... ]
            },
            "subServiceIds": ["uuid1", "uuid2"]
          }
        ],
        "availability": [
          {
            "id": "uuid",
            "dayOfWeek": 1,
            "startTime": "09:00",
            "endTime": "17:00",
            "isAvailable": true
          }
        ],
        "reviews": [
          {
            "id": "uuid",
            "rating": 5,
            "comment": "Great service!",
            "user": {
              "id": "uuid",
              "name": "John Doe",
              "profileImage": "/uploads/images/profile.jpg"
            },
            "createdAt": "2024-01-01T00:00:00.000Z"
          }
        ],
        "distance": 2.5,  // in kilometers, only if user has location set
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ]
  }
}
```

**Error Responses:**
- `400 Bad Request`: User location not set (if radius filter requested)
- `401 Unauthorized`: Invalid or missing token

---

### GET `/api/customer/service-provider/:id`
Get detailed service provider information with distance calculation.

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "description": "Professional cleaning services",
    "portfolioImages": ["/uploads/images/img1.jpg", "/uploads/images/img2.jpg"],
    "status": "ACTIVE",
    "user": {
      "id": "uuid",
      "name": "Clean Pro",
      "email": "clean@example.com",
      "mobileNumber": "+1234567890",
      "whatsappNumber": "+1234567890",
      "country": "USA",
      "city": "New York",
      "profileImage": "/uploads/images/profile.jpg",
      "latitude": 40.7128,
      "longitude": -74.0060
    },
    "services": [ ... ],  // Full service details with categories and sub-services
    "availability": [ ... ],  // All availability slots
    "reviews": [ ... ],  // All reviews with user details
    "isFavorited": false,  // Whether current user has favorited this provider
    "distance": 2.5  // Distance in kilometers (null if locations not set)
  }
}
```

**Error Responses:**
- `404 Not Found`: Service provider not found
- `401 Unauthorized`: Invalid or missing token

---

### POST `/api/customer/favorite/:serviceProviderId`
Add service provider to favorites (if not already favorited).

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `serviceProviderId` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Added to favorites",
  "isFavorited": true
}
```

**Error Responses:**
- `404 Not Found`: Service provider not found

---

### DELETE `/api/customer/favorite/:serviceProviderId`
Remove service provider from favorites.

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `serviceProviderId` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Removed from favorites",
  "isFavorited": false
}
```

---

### GET `/api/customer/favorites`
Get all favorited service providers.

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | String | No | Search query (searches provider name, description, category, service) |

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "userId": "uuid",
      "serviceProviderId": "uuid",
      "serviceProvider": {
        "id": "uuid",
        "status": "ACTIVE",
        "description": "Professional cleaning services",
        "portfolioImages": ["/uploads/images/img1.jpg"],
        "user": {
          "id": "uuid",
          "name": "Clean Pro",
          "email": "clean@example.com",
          "mobileNumber": "+1234567890",
          "whatsappNumber": "+1234567890",
          "country": "USA",
          "city": "New York",
          "profileImage": "/uploads/images/profile.jpg",
          "latitude": 40.7128,
          "longitude": -74.0060
        },
        "services": [ ... ],
        "availability": [ ... ],
        "reviews": [ ... ],
        "distance": 2.5  // in kilometers, if user location is set
      },
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

### POST `/api/customer/review/:serviceProviderId`
Add or update a review for a service provider (one review per user per provider).

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `serviceProviderId` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:**
```json
{
  "rating": 5,              // Required: integer, must be between 1 and 5
  "comment": "Great service!"  // Optional: string
}
```

**Response (201 Created or 200 OK):**
```json
{
  "success": true,
  "message": "Review added successfully",  // or "Review updated"
  "data": {
    "id": "uuid",
    "userId": "uuid",
    "serviceProviderId": "uuid",
    "rating": 5,
    "comment": "Great service!",
    "user": {
      "id": "uuid",
      "name": "John Doe",
      "profileImage": "/uploads/images/profile.jpg"
    },
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: Invalid rating (must be 1-5)
- `404 Not Found`: Service provider not found

---

### POST `/api/customer/report/:serviceProviderId`
Report a service provider (changes provider status to REPORTED).

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `serviceProviderId` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:**
```json
{
  "reason": "Inappropriate behavior"  // Optional: string
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Service provider reported successfully"
}
```

---

### PUT `/api/customer/profile`
Update customer profile information.

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `profileImage` | File | No | Profile image (JPEG, PNG, GIF, WebP, max 10MB) |
| `name` | String | No | Customer name |
| `email` | String | No | Email address (must be unique if changed) |
| `mobileNumber` | String | No | Mobile phone number |
| `country` | String | No | Country name |
| `city` | String | No | City name |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "user": {
    "id": "uuid",
    "email": "customer@example.com",
    "name": "John Doe",
    "mobileNumber": "+1234567890",
    "country": "USA",
    "city": "New York",
    "profileImage": "/uploads/images/profile.jpg",
    "customerProfile": { ... },
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: Email already exists (if email is being changed)

---

### GET `/api/customer/privacy-policy`
Get privacy policy PDF information.

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "pdfUrl": "/uploads/pdfs/policy.pdf",
    "version": 1,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `404 Not Found`: Privacy policy not found

---

### GET `/api/customer/help-center`
Get help center contact information.

**Authentication:** Required (Customer)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "email": "support@gfinder.com",
    "phone": "+1234567890",
    "whatsapp": "+1234567890"
  }
}
```

---

## 🛠️ Service Provider APIs (`/api/service-provider`)

**All endpoints require:** Service Provider authentication (Bearer token)
**Base Path:** `/api/service-provider`

### GET `/api/service-provider/home`
Get marketplace view (same as customer home screen).

**Authentication:** Required (Service Provider)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:** Same as `/api/customer/home`
- `categoryId` (String, optional)
- `subServiceId` (String, optional)
- `radius` (Number, optional, default: 1)
- `search` (String, optional)

**Request Body:** None

**Response:** Same format as `/api/customer/home`

---

### PUT `/api/service-provider/location`
Update service provider location.

**Authentication:** Required (Service Provider)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "latitude": 40.7128,   // Required: number, valid latitude
  "longitude": -74.0060  // Required: number, valid longitude
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Location updated successfully",
  "user": {
    "latitude": 40.7128,
    "longitude": -74.0060
  }
}
```

---

### GET `/api/service-provider/portfolio`
Get service provider's own portfolio details.

**Authentication:** Required (Service Provider)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | String | No | Search within portfolio services |

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "ACTIVE",
    "description": "Professional cleaning services",
    "portfolioImages": ["/uploads/images/img1.jpg", "/uploads/images/img2.jpg"],
    "user": {
      "id": "uuid",
      "name": "Clean Pro",
      "email": "clean@example.com",
      "mobileNumber": "+1234567890",
      "whatsappNumber": "+1234567890",
      "country": "USA",
      "city": "New York",
      "profileImage": "/uploads/images/profile.jpg"
    },
    "services": [ ... ],  // All services with categories and sub-services
    "availability": [ ... ],  // All availability slots
    "reviews": [ ... ],  // All reviews
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z",
    "approvedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `404 Not Found`: Service provider profile not found

---

### PUT `/api/service-provider/portfolio/images`
Update portfolio images (maximum 5 images total).

**Authentication:** Required (Service Provider)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `portfolioImages` | Files | No | Image files (JPEG, PNG, GIF, WebP, max 5 total, max 10MB each) |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Portfolio images updated successfully",
  "data": {
    "id": "uuid",
    "portfolioImages": [
      "/uploads/images/img1.jpg",
      "/uploads/images/img2.jpg",
      "/uploads/images/img3.jpg"
    ]
  }
}
```

**Error Responses:**
- `400 Bad Request`: More than 5 images attempted

---

### PUT `/api/service-provider/profile`
Update service provider profile.

**Authentication:** Required (Service Provider)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `profileImage` | File | No | Profile image (JPEG, PNG, GIF, WebP, max 10MB) |
| `name` | String | No | Service provider name |
| `email` | String | No | Email address (must be unique if changed) |
| `mobileNumber` | String | No | Mobile phone number |
| `whatsappNumber` | String | No | WhatsApp number with country code |
| `country` | String | No | Country name |
| `city` | String | No | City name |
| `description` | String | No | Service description |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "data": {
    "id": "uuid",
    "status": "ACTIVE",
    "description": "Professional cleaning services",
    "user": { ... },
    "services": [ ... ],
    "availability": [ ... ]
  }
}
```

**Error Responses:**
- `400 Bad Request`: Email already exists (if email is being changed)

---

### PUT `/api/service-provider/availability`
Update availability timing for each day of the week.

**Authentication:** Required (Service Provider)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:**
```json
{
  "availability": [  // Required: array of availability objects
    {
      "dayOfWeek": 1,           // Required: integer (0=Sunday, 1=Monday, ..., 6=Saturday)
      "startTime": "09:00",     // Required: string, format "HH:MM" (24-hour)
      "endTime": "17:00",       // Required: string, format "HH:MM" (24-hour)
      "isAvailable": true       // Optional: boolean, defaults to true
    },
    {
      "dayOfWeek": 2,
      "startTime": "09:00",
      "endTime": "17:00",
      "isAvailable": true
    }
    // ... more days
  ]
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Availability updated successfully",
  "data": [
    {
      "id": "uuid",
      "serviceProviderId": "uuid",
      "dayOfWeek": 1,
      "startTime": "09:00",
      "endTime": "17:00",
      "isAvailable": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

**Error Responses:**
- `400 Bad Request`: Invalid array format or missing required fields

---

### GET `/api/service-provider/privacy-policy`
Get privacy policy PDF information.

**Authentication:** Required (Service Provider)

**Response:** Same format as customer endpoint

---

### GET `/api/service-provider/help-center`
Get help center contact information.

**Authentication:** Required (Service Provider)

**Response:** Same format as customer endpoint

---

## 👨‍💼 Admin APIs (`/api/admin`)

**All endpoints require:** Admin authentication (Bearer token)
**Base Path:** `/api/admin`

### GET `/api/admin/dashboard`
Get dashboard statistics and analytics.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "activeCount": 150,        // Number of active service providers
    "pendingCount": 25,        // Number of pending approvals
    "reportedCount": 5,        // Number of reported providers
    "inactiveCount": 10,       // Number of inactive/blocked providers
    "countryDistribution": {   // Count of active providers by country
      "Pakistan": 80,
      "USA": 50,
      "India": 20,
      "UK": 15
    }
  }
}
```

---

### GET `/api/admin/service-providers`
Get all service providers with optional filters.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | String | No | Filter by status: PENDING, ACTIVE, INACTIVE, REPORTED, REJECTED |
| `search` | String | No | Search query (searches name, email, description, category, service) |
| `radius` | Number | No | Filter by radius in kilometers (requires latitude/longitude) |
| `latitude` | Number | No | User latitude for radius filter |
| `longitude` | Number | No | User longitude for radius filter |

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "status": "ACTIVE",
      "description": "Professional cleaning services",
      "portfolioImages": ["/uploads/images/img1.jpg"],
      "user": { ... },
      "services": [ ... ],
      "availability": [ ... ],
      "reviews": [ ... ],
      "distance": 2.5,  // if radius filter applied
      "createdAt": "2024-01-01T00:00:00.000Z",
      "approvedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

### GET `/api/admin/service-providers/pending`
Get all pending service provider requests.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:** None

**Response:** Same format as `/api/admin/service-providers`, filtered to status: PENDING

---

### GET `/api/admin/service-providers/reported`
Get all reported service providers.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:** None

**Response:** Same format as above, includes `reports` array with report details:
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "status": "REPORTED",
      "user": { ... },
      "services": [ ... ],
      "reports": [
        {
          "id": "uuid",
          "reason": "Inappropriate behavior",
          "user": {
            "id": "uuid",
            "name": "John Doe"
          },
          "createdAt": "2024-01-01T00:00:00.000Z"
        }
      ]
    }
  ]
}
```

---

### GET `/api/admin/service-providers/:id`
Get detailed service provider information.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response:** Complete service provider data with all relations including reports (if any)

---

### PUT `/api/admin/service-providers/:id/approve`
Approve a pending service provider (changes status to ACTIVE).

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Service provider approved successfully",
  "data": {
    "id": "uuid",
    "status": "ACTIVE",
    "approvedAt": "2024-01-01T00:00:00.000Z",
    "user": { ... }
  }
}
```

---

### PUT `/api/admin/service-providers/:id/reject`
Reject a pending service provider (changes status to REJECTED).

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Service provider rejected",
  "data": {
    "id": "uuid",
    "status": "REJECTED",
    "user": { ... }
  }
}
```

---

### PUT `/api/admin/service-providers/:id/status`
Update service provider status (ACTIVE, INACTIVE, or REJECTED).

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Service provider profile ID (UUID) |

**Query Parameters:** None

**Request Body:**
```json
{
  "status": "ACTIVE"  // Required: string, must be "ACTIVE", "INACTIVE", or "REJECTED"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Service provider status updated to ACTIVE",
  "data": {
    "id": "uuid",
    "status": "ACTIVE",
    "user": { ... }
  }
}
```

**Error Responses:**
- `400 Bad Request`: Invalid status value

---

### GET `/api/admin/categories`
Get all categories with their services and sub-services.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Cleaning",
      "icon": "/uploads/images/icon.png",
      "services": [
        {
          "id": "uuid",
          "title": "Home Cleaning",
          "icon": "/uploads/images/service-icon.png",
          "subServices": [
            {
              "id": "uuid",
              "title": "Deep Cleaning",
              "createdAt": "2024-01-01T00:00:00.000Z"
            }
          ]
        }
      ],
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

### POST `/api/admin/categories`
Create a new category.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | String | **Yes** | Category title (must be unique) |
| `icon` | File | No | Category icon image (JPEG, PNG, GIF, WebP, max 10MB) |

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Category created successfully",
  "data": {
    "id": "uuid",
    "title": "Cleaning",
    "icon": "/uploads/images/icon.png",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: Title already exists

---

### PUT `/api/admin/categories/:id`
Update a category.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Category ID (UUID) |

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | String | No | Category title (must be unique if changed) |
| `icon` | File | No | Category icon image |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Category updated successfully",
  "data": { ... }
}
```

---

### DELETE `/api/admin/categories/:id`
Delete a category (also deletes associated services and sub-services).

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Category ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Category deleted successfully"
}
```

---

### GET `/api/admin/services`
Get all services with their categories and sub-services.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `categoryId` | String | No | Filter by category ID |

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Home Cleaning",
      "icon": "/uploads/images/service-icon.png",
      "categoryId": "uuid",
      "category": {
        "id": "uuid",
        "title": "Cleaning"
      },
      "subServices": [ ... ],
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

### POST `/api/admin/services`
Create a new service.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | String | **Yes** | Service title |
| `categoryId` | String | **Yes** | Category ID (UUID) |
| `icon` | File | No | Service icon image (JPEG, PNG, GIF, WebP, max 10MB) |

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Service created successfully",
  "data": {
    "id": "uuid",
    "title": "Home Cleaning",
    "icon": "/uploads/images/service-icon.png",
    "categoryId": "uuid",
    "category": { ... },
    "subServices": [],
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

---

### PUT `/api/admin/services/:id`
Update a service.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Service ID (UUID) |

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | String | No | Service title |
| `categoryId` | String | No | Category ID (UUID) |
| `icon` | File | No | Service icon image |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Service updated successfully",
  "data": { ... }
}
```

---

### DELETE `/api/admin/services/:id`
Delete a service (also deletes associated sub-services).

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Service ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Service deleted successfully"
}
```

---

### GET `/api/admin/sub-services`
Get all sub-services with their parent services.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:** None

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `serviceId` | String | No | Filter by service ID |

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Deep Cleaning",
      "serviceId": "uuid",
      "service": {
        "id": "uuid",
        "title": "Home Cleaning",
        "category": {
          "id": "uuid",
          "title": "Cleaning"
        }
      },
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

### POST `/api/admin/sub-services`
Create a new sub-service.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:**
```json
{
  "title": "Deep Cleaning",    // Required: string, must be unique within the service
  "serviceId": "uuid"          // Required: string, UUID of parent service
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Sub-service created successfully",
  "data": {
    "id": "uuid",
    "title": "Deep Cleaning",
    "serviceId": "uuid",
    "service": { ... },
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: Sub-service title already exists for this service

---

### PUT `/api/admin/sub-services/:id`
Update a sub-service.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Sub-service ID (UUID) |

**Query Parameters:** None

**Request Body:**
```json
{
  "title": "Deep Cleaning",    // Optional: string
  "serviceId": "uuid"          // Optional: string, UUID of parent service
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Sub-service updated successfully",
  "data": { ... }
}
```

---

### DELETE `/api/admin/sub-services/:id`
Delete a sub-service.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | String | **Yes** | Sub-service ID (UUID) |

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Sub-service deleted successfully"
}
```

---

### GET `/api/admin/privacy-policy`
Get privacy policy PDF information.

**Authentication:** Required (Admin)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "pdfUrl": "/uploads/pdfs/policy.pdf",
    "version": 1,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

---

### PUT `/api/admin/privacy-policy`
Update privacy policy PDF (creates new version).

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pdf` | File | **Yes** | PDF file (max 10MB) |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Privacy policy updated successfully",
  "data": {
    "id": "uuid",
    "pdfUrl": "/uploads/pdfs/policy-v2.pdf",
    "version": 2,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `400 Bad Request`: PDF file is required

---

### GET `/api/admin/settings`
Get app settings (support contact information).

**Authentication:** Required (Admin)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "supportEmail": "support@gfinder.com",
    "supportPhone": "+1234567890",
    "supportWhatsapp": "+1234567890",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

---

### PUT `/api/admin/settings`
Update app settings (support contacts).

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body:**
```json
{
  "supportEmail": "support@gfinder.com",    // Optional: string, valid email format
  "supportPhone": "+1234567890",            // Optional: string
  "supportWhatsapp": "+1234567890"          // Optional: string
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "App settings updated successfully",
  "data": {
    "id": "uuid",
    "supportEmail": "support@gfinder.com",
    "supportPhone": "+1234567890",
    "supportWhatsapp": "+1234567890",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

---

### PUT `/api/admin/profile`
Update admin profile.

**Authentication:** Required (Admin)

**Request Headers:**
```
Authorization: Bearer <token>
Content-Type: multipart/form-data
```

**Path Parameters:** None

**Query Parameters:** None

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `profileImage` | File | No | Profile image (JPEG, PNG, GIF, WebP, max 10MB) |
| `name` | String | No | Admin name |
| `email` | String | No | Email address (must be unique if changed) |
| `mobileNumber` | String | No | Mobile phone number |
| `country` | String | No | Country name |
| `city` | String | No | City name |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Profile updated successfully",
  "user": {
    "id": "uuid",
    "email": "admin@gfinder.com",
    "name": "Admin User",
    "role": "ADMIN",
    "mobileNumber": "+1234567890",
    "country": "USA",
    "city": "New York",
    "profileImage": "/uploads/images/profile.jpg",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

---

## 🌐 Public APIs (`/api`)

### GET `/api/health`
Health check endpoint (no authentication required).

**Request Headers:** None

**Path Parameters:** None

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "status": "OK",
  "message": "GFinder API is running"
}
```

---

### GET `/api/privacy-policy`
Get privacy policy PDF information (public, no authentication required).

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "pdfUrl": "/uploads/pdfs/policy.pdf",
    "version": 1,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Error Responses:**
- `404 Not Found`: Privacy policy not found

---

### GET `/api/help-center`
Get help center contact information (public, no authentication required).

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "email": "support@gfinder.com",
    "phone": "+1234567890",
    "whatsapp": "+1234567890"
  }
}
```

---

### GET `/api/categories`
Get all categories with services and sub-services (public, no authentication required).

**Request Headers:** None

**Path Parameters:** None

**Query Parameters:** None

**Request Body:** None

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Cleaning",
      "icon": "/uploads/images/icon.png",
      "services": [
        {
          "id": "uuid",
          "title": "Home Cleaning",
          "icon": "/uploads/images/service-icon.png",
          "subServices": [
            {
              "id": "uuid",
              "title": "Deep Cleaning",
              "createdAt": "2024-01-01T00:00:00.000Z"
            }
          ]
        }
      ],
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

## 📝 Response Format

All API responses follow a consistent format:

### Success Response
```json
{
  "success": true,
  "message": "Operation successful",  // Optional: string
  "data": { ... }  // or array, depending on endpoint
}
```

### Error Response
```json
{
  "success": false,
  "message": "Error message here"
}
```

---

## ⚠️ HTTP Status Codes

- `200 OK` - Successful GET, PUT, DELETE requests
- `201 Created` - Successful POST requests (resource created)
- `400 Bad Request` - Invalid request data, missing required fields, validation errors
- `401 Unauthorized` - Missing or invalid authentication token
- `403 Forbidden` - Insufficient permissions (wrong user role)
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

---

## ⚠️ Common Error Messages

- `"Authentication required"` - Missing or invalid JWT token
- `"Invalid email or password"` - Login credentials incorrect
- `"Email already registered"` - Email exists during signup
- `"User not found"` - User ID doesn't exist
- `"Insufficient permissions"` - User doesn't have required role (CUSTOMER, SERVICE_PROVIDER, ADMIN)
- `"Invalid or expired OTP"` - OTP code incorrect or expired (10 minute expiry)
- `"Service provider not found"` - Service provider ID doesn't exist
- `"Category title already exists"` - Category title must be unique
- `"Sub-service with this title already exists for this service"` - Sub-service title must be unique within a service

---

## 📤 File Upload Specifications

### Supported File Types
- **Images**: JPEG, JPG, PNG, GIF, WebP
- **PDFs**: PDF files only

### File Size Limits
- Maximum file size: 10MB per file (configurable via `MAX_FILE_SIZE` in `.env`)
- Portfolio images: Maximum 5 images total per service provider

### Upload Endpoints
All file uploads use `multipart/form-data` content type:
- Profile images: `profileImage` field (single file)
- Portfolio images: `portfolioImages` field (multiple files, max 5)
- Service/Category icons: `icon` field (single file)
- PDF files: `pdf` field (single file)

### File URLs
Uploaded files are accessible at:
```
http://localhost:3000/uploads/images/filename.jpg
http://localhost:3000/uploads/pdfs/filename.pdf
```

---

## 🔑 Authentication Flow

1. **Signup/Login**: Receive JWT token in response
2. **Store Token**: Save token in client storage (localStorage, secure storage, etc.)
3. **Include in Requests**: Add `Authorization: Bearer <token>` header to all authenticated requests
4. **Token Expiry**: 
   - Default: 7 days
   - With `rememberMe=true`: 30 days
5. **Refresh**: Login again to get new token
6. **Logout**: Remove token from client storage (backend doesn't maintain token blacklist)

---

## 📊 Data Models & Enums

### User Roles
- `CUSTOMER` - Regular customer account
- `SERVICE_PROVIDER` - Service provider account
- `ADMIN` - Administrator account

### Service Provider Status
- `PENDING` - Awaiting admin approval (default for new signups)
- `ACTIVE` - Approved and active (visible to customers)
- `INACTIVE` - Blocked/deactivated by admin
- `REPORTED` - Reported by users (requires admin review)
- `REJECTED` - Rejected by admin

### Language
- `ENGLISH`
- `URDU`

### Availability Days (dayOfWeek)
- `0` - Sunday
- `1` - Monday
- `2` - Tuesday
- `3` - Wednesday
- `4` - Thursday
- `5` - Friday
- `6` - Saturday

### Time Format
- Format: `"HH:MM"` (24-hour format)
- Example: `"09:00"` (9:00 AM), `"17:00"` (5:00 PM)

---

## 🔍 Search Functionality

Search is implemented across multiple endpoints and searches within:
- Service provider names
- Category titles
- Service titles
- Sub-service titles
- Descriptions

**Search behavior:**
- Case-insensitive
- Partial match (contains)
- Searches across all specified fields simultaneously (OR logic)

---

## 📍 Location-Based Features

### Distance Calculation
Uses Haversine formula to calculate distance between coordinates in kilometers.

### Radius Filtering
- **Default radius**: 1 km
- **Configurable**: Via `radius` query parameter (in kilometers)
- **Requires**: User location (latitude/longitude) must be set
- **Returns**: Distance in kilometers for each result
- **Sorting**: Results sorted by distance (closest first)

### Location Update
Customers and service providers must update their location for:
- Filtering service providers by radius
- Showing distance to service providers
- Home screen listings (default shows providers within 1000m if location set)

**Location Format:**
- Latitude: Number between -90 and 90
- Longitude: Number between -180 and 180

---

## 📱 Integration Examples

### React/Next.js Example
```javascript
// Login
const login = async (email, password) => {
  const response = await fetch('http://localhost:3000/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });
  const data = await response.json();
  if (data.success) {
    localStorage.setItem('token', data.token);
    localStorage.setItem('user', JSON.stringify(data.user));
  }
  return data;
};

// Authenticated Request - Get Home Screen
const getHomeScreen = async (radius = 1, search = '') => {
  const token = localStorage.getItem('token');
  const queryParams = new URLSearchParams({
    radius: radius.toString(),
    ...(search && { search })
  });
  
  const response = await fetch(
    `http://localhost:3000/api/customer/home?${queryParams}`,
    {
      headers: { 'Authorization': `Bearer ${token}` }
    }
  );
  return await response.json();
};

// File Upload - Update Profile
const updateProfile = async (imageFile, name) => {
  const token = localStorage.getItem('token');
  const formData = new FormData();
  if (imageFile) formData.append('profileImage', imageFile);
  if (name) formData.append('name', name);
  
  const response = await fetch('http://localhost:3000/api/customer/profile', {
    method: 'PUT',
    headers: { 'Authorization': `Bearer ${token}` },
    body: formData
  });
  return await response.json();
};

// Add Review
const addReview = async (serviceProviderId, rating, comment) => {
  const token = localStorage.getItem('token');
  const response = await fetch(
    `http://localhost:3000/api/customer/review/${serviceProviderId}`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ rating, comment })
    }
  );
  return await response.json();
};
```

### cURL Examples
```bash
# Login
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@gfinder.com","password":"Admin@123"}'

# Get Home Screen (with token)
curl http://localhost:3000/api/customer/home?radius=5 \
  -H "Authorization: Bearer YOUR_TOKEN"

# Update Profile with Image
curl -X PUT http://localhost:3000/api/customer/profile \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "profileImage=@/path/to/image.jpg" \
  -F "name=John Doe"

# Add Favorite
curl -X POST http://localhost:3000/api/customer/favorite/SERVICE_PROVIDER_ID \
  -H "Authorization: Bearer YOUR_TOKEN"

# Add Review
curl -X POST http://localhost:3000/api/customer/review/SERVICE_PROVIDER_ID \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"rating":5,"comment":"Great service!"}'
```

---

## 🔄 Rate Limiting

Currently, rate limiting is not implemented. Consider implementing in production:
- Login attempts: 5 per 15 minutes per IP
- OTP requests: 10 per hour per email
- API requests: 100 per minute per IP

---

## 📝 Important Notes

1. **Service Provider Status**: New service providers start as `PENDING` and require admin approval to become `ACTIVE`
2. **Reviews**: Users can only have one review per service provider (updates existing if present)
3. **Favorites**: Users can favorite/unfavorite service providers (toggle endpoint)
4. **Reports**: Reporting a service provider automatically changes status to `REPORTED`
5. **Portfolio Images**: Maximum 5 images per service provider (oldest removed if exceeded)
6. **Location**: Required for filtering and distance calculations (update via location endpoint)
7. **OTP Expiry**: OTP codes expire after 10 minutes
8. **File Storage**: Files are stored locally in `uploads/` directory (consider cloud storage for production)
9. **Email**: OTP emails are sent but failures don't block the request (check logs for email errors)
10. **JSON Fields**: Some endpoints accept JSON strings or arrays for fields like `serviceIds`, `availability` (for form data compatibility)

---

## 🚀 Quick Reference

### Most Used Customer Endpoints
1. `POST /api/auth/login` - Login
2. `PUT /api/customer/location` - Set location (required for filtering)
3. `GET /api/customer/home?radius=5&search=cleaning` - Get service providers
4. `GET /api/customer/service-provider/:id` - View provider details
5. `POST /api/customer/favorite/:id` - Add to favorites
6. `POST /api/customer/review/:id` - Add review

### Most Used Admin Endpoints
1. `GET /api/admin/dashboard` - View statistics
2. `GET /api/admin/service-providers/pending` - Review pending requests
3. `PUT /api/admin/service-providers/:id/approve` - Approve provider
4. `GET /api/admin/categories` - Manage categories
5. `POST /api/admin/categories` - Create category

---

**For setup instructions, see [SETUP.md](./SETUP.md)**
**For general information, see [README.md](./README.md)**
