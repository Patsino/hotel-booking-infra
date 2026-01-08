# API Reference

## Overview

The Hotel Booking Platform consists of four microservices, each with its own API:

| Service | Base URL (Local) | Base URL (Azure) |
|---------|-----------------|------------------|
| Users | `http://localhost:8081/api` | `https://hotel-booking-users-api-*.azurewebsites.net/api` |
| Hotels | `http://localhost:8082/api` | `https://hotel-booking-hotels-api-*.azurewebsites.net/api` |
| Reservations | `http://localhost:8083/api` | `https://hotel-booking-reservations-api-*.azurewebsites.net/api` |
| Payments | `http://localhost:8084/api` | `https://hotel-booking-payments-api-*.azurewebsites.net/api` |

**Authentication:** Bearer Token (JWT)

**Format:** All requests and responses use JSON

---

## Users Service

### Authentication

#### POST /auth/register

Register a new user account.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "Password123!",
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| email | string | Yes | Valid email address |
| password | string | Yes | Minimum 8 characters |


**Response:** `201 Created`
```json
{
  "id": "guid",
  "email": "user@example.com",
}
```

---

#### POST /auth/login

Authenticate and receive tokens.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "Password123!"
}
```

**Response:** `200 OK`
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
  "expiresAt": "2026-01-05T14:00:00Z"
}
```

---

#### POST /auth/refresh

Refresh access token using refresh token.

**Request Body:**
```json
{
  "refreshToken": "current-refresh-token"
}
```

**Response:** `200 OK`
```json
{
  "accessToken": "new-access-token",
  "refreshToken": "new-refresh-token",
  "expiresAt": "2026-01-05T15:00:00Z"
}
```

---

#### POST /auth/logout

Revoke refresh token.

**Headers:** `Authorization: Bearer <access-token>`

**Request Body:**
```json
{
  "refreshToken": "refresh-token-to-revoke"
}
```

**Response:** `200 OK`

---

### Users

#### GET /users/me

Get current user profile.

**Headers:** `Authorization: Bearer <access-token>`

**Response:** `200 OK`
```json
{
  "id": "guid",
  "email": "user@example.com",
  "role": "User",
  "createdAt": "2026-01-01T00:00:00Z"
}
```

---

#### GET /users/{id}

Get user by ID (Admin only).

**Headers:** `Authorization: Bearer <access-token>`

**Response:** `200 OK`

---

## Hotels Service

### Hotels

#### GET /hotels

Search for hotels.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| location | string | No | Filter by city/location |
| checkIn | date | No | Check-in date (YYYY-MM-DD) |
| checkOut | date | No | Check-out date (YYYY-MM-DD) |
| guests | integer | No | Number of guests |
| page | integer | No | Page number (default: 1) |
| pageSize | integer | No | Items per page (default: 10) |

**Response:** `200 OK`
```json
{
  "items": [
    {
      "id": "guid",
      "name": "Grand Hotel",
      "location": "Paris, France",
      "description": "Luxury hotel in city center",
      "rating": 4.5,
      "minPrice": 150.00,
      "imageUrl": "https://..."
    }
  ],
  "totalCount": 50,
  "page": 1,
  "pageSize": 10
}
```

---

#### GET /hotels/{id}

Get hotel details with rooms.

**Response:** `200 OK`
```json
{
  "id": "guid",
  "name": "Grand Hotel",
  "location": "Paris, France",
  "description": "Luxury hotel...",
  "cancelFreeDaysBefore": 3,
  "ownerId": "guid",
  "isApproved": true,
  "rooms": [
    {
      "id": "guid",
      "roomNumber": "101",
      "roomType": "Deluxe",
      "pricePerNight": 200.00,
      "maxGuests": 2,
      "description": "King bed with city view"
    }
  ]
}
```

---

#### POST /hotels

Create a new hotel (HotelOwner only).

**Headers:** `Authorization: Bearer <access-token>`

**Request Body:**
```json
{
  "name": "My Hotel",
  "location": "London, UK",
  "description": "Boutique hotel in Soho",
  "cancelFreeDaysBefore": 5
}
```

**Response:** `201 Created`

---

#### PUT /hotels/{id}

Update hotel details (Owner only).

---

#### DELETE /hotels/{id}

Delete hotel (Owner only).

---

#### GET /hotels/{id}/reservations

Get all reservations for a hotel (Owner or Admin only).

**Headers:** `Authorization: Bearer <access-token>`

**Response:** `200 OK`
```json
[
  {
    "id": 123,
    "userId": 42,
    "roomId": 55,
    "roomNumber": "101",
    "startDate": "2024-12-20",
    "endDate": "2024-12-22",
    "guestsCount": 2,
    "guestsNames": "John Doe, Jane Smith",
    "status": "Confirmed",
    "cancellationStatus": "None",
    "cancellationReason": null,
    "cancellationRequestedAt": null,
    "createdAt": "2024-12-15T10:30:00Z"
  }
]
```

| Field | Description |
|-------|-------------|
| status | Pending, Held, Confirmed, Canceled |
| cancellationStatus | None, Requested, AutoCanceled, AdminApproved, AdminRejected |

**Response Codes:**
- `200 OK` - Reservations returned
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Not owner or admin
- `404 Not Found` - Hotel not found

---

#### POST /hotels/{id}/approve

Approve hotel (Admin only).

**Headers:** `Authorization: Bearer <access-token>`

**Response:** `200 OK`

---

### Rooms

#### POST /hotels/{hotelId}/rooms

Add room to hotel (Owner only).

**Request Body:**
```json
{
  "roomNumber": "102",
  "roomType": "Suite",
  "pricePerNight": 350.00,
  "maxGuests": 4,
  "description": "Two bedroom suite"
}
```

---

#### PUT /hotels/{hotelId}/rooms/{roomId}

Update room details.

---

#### DELETE /hotels/{hotelId}/rooms/{roomId}

Remove room from hotel.

---

#### GET /hotels/{hotelId}/rooms/{roomId}/availability

Check room availability for dates.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| checkIn | date | Yes | Check-in date |
| checkOut | date | Yes | Check-out date |

**Response:** `200 OK`
```json
{
  "isAvailable": true,
  "roomId": "guid",
  "checkIn": "2026-02-01",
  "checkOut": "2026-02-05"
}
```

---

## Reservations Service

### Reservations

#### GET /reservations

Get user's reservations.

**Headers:** `Authorization: Bearer <access-token>`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| status | string | Filter by status |
| page | integer | Page number |

**Response:** `200 OK`
```json
{
  "items": [
    {
      "id": "guid",
      "hotelId": "guid",
      "hotelName": "Grand Hotel",
      "roomId": "guid",
      "roomNumber": "101",
      "checkIn": "2026-02-01",
      "checkOut": "2026-02-05",
      "guestCount": 2,
      "totalPrice": 800.00,
      "status": "Confirmed"
    }
  ]
}
```

---

#### GET /reservations/{id}

Get reservation details.

---

#### POST /reservations

Create a new reservation.

**Headers:** `Authorization: Bearer <access-token>`

**Request Body:**
```json
{
  "hotelId": "guid",
  "roomId": "guid",
  "checkIn": "2026-02-01",
  "checkOut": "2026-02-05",
  "guestCount": 2
}
```

**Response:** `201 Created`
```json
{
  "id": "guid",
  "status": "Pending",
  "totalPrice": 800.00
}
```

---

#### POST /reservations/{id}/cancel

Request reservation cancellation.

**Headers:** `Authorization: Bearer <access-token>`

**Response:** `200 OK`
```json
{
  "id": "guid",
  "status": "Cancelled",
  "cancellationType": "FreeCancellation"
}
```

Or for late cancellations:
```json
{
  "id": "guid",
  "status": "CancellationPending",
  "cancellationType": "RequiresApproval"
}
```

---

#### POST /reservations/{id}/approve-cancellation

Approve cancellation (Admin only).

---

#### POST /reservations/{id}/reject-cancellation

Reject cancellation (Admin only).

---

## Payments Service

### Payments

#### POST /payments/create-intent

Create a Stripe payment intent.

**Headers:** `Authorization: Bearer <access-token>`

**Request Body:**
```json
{
  "reservationId": "guid",
  "amount": 800.00,
  "currency": "usd"
}
```

**Response:** `200 OK`
```json
{
  "paymentId": "guid",
  "clientSecret": "pi_xxx_secret_xxx",
  "amount": 800.00,
  "currency": "usd",
  "status": "RequiresPayment"
}
```

---

#### POST /payments/{id}/confirm

Confirm payment after Stripe processing.

**Request Body:**
```json
{
  "paymentMethodId": "pm_card_visa"
}
```

**Response:** `200 OK`
```json
{
  "paymentId": "guid",
  "status": "Succeeded",
  "paidAt": "2026-01-05T12:00:00Z"
}
```

---

#### GET /payments/{id}

Get payment details.

---

#### POST /payments/{id}/refund

Process refund (Admin or System).

**Request Body:**
```json
{
  "reason": "Cancellation within free period"
}
```

**Response:** `200 OK`
```json
{
  "refundId": "guid",
  "amount": 800.00,
  "status": "Succeeded"
}
```

---

#### POST /payments/webhook

Stripe webhook endpoint for payment events.

**Note:** This endpoint is called by Stripe, not by clients directly.

---

## Error Responses

All services return errors in a consistent format:

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "Bad Request",
  "status": 400,
  "detail": "Validation failed",
  "errors": {
    "email": ["The Email field is required."]
  }
}
```

| Status Code | Description |
|-------------|-------------|
| 400 | Bad Request - Invalid input data |
| 401 | Unauthorized - Missing or invalid token |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Resource already exists or state conflict |
| 500 | Internal Server Error - Unexpected error |

## Swagger/OpenAPI

Interactive API documentation is available at each service's `/swagger` endpoint:

- Users: `http://localhost:8081/swagger`
- Hotels: `http://localhost:8082/swagger`
- Reservations: `http://localhost:8083/swagger`
- Payments: `http://localhost:8084/swagger`
---

## Internal Service-to-Service Endpoints

These endpoints are used for inter-service communication and require API key authentication via `X-API-Key` header.

### Reservations Service - Internal

#### POST /internal/reservations/batch-availability

Check availability for multiple rooms in a date range.

**Headers:** `X-API-Key: <service-api-key>`

**Request Body:**
```json
{
  "roomIds": [1, 2, 3],
  "startDate": "2024-12-20",
  "endDate": "2024-12-25"
}
```

**Response:** `200 OK`
```json
{
  "unavailableRoomIds": [2]
}
```

---

#### POST /internal/reservations/by-rooms

Get all reservations for specified rooms (used by Hotels Service for owner view).

**Headers:** `X-API-Key: <service-api-key>`

**Request Body:**
```json
{
  "roomIds": [1, 2, 3]
}
```

**Response:** `200 OK`
```json
[
  {
    "id": 123,
    "userId": 42,
    "roomId": 1,
    "startDate": "2024-12-20",
    "endDate": "2024-12-22",
    "guestsCount": 2,
    "guestsNames": "John Doe",
    "status": "Confirmed",
    "cancellationStatus": "None",
    "createdAt": "2024-12-15T10:30:00Z"
  }
]
```
