# Complete Hotel Booking Flow Tutorial

## Overview

This tutorial walks you through the complete end-to-end booking process, from user registration to payment confirmation. You'll learn how to integrate all four microservices to create a seamless booking experience.

## Prerequisites

- All services running (see [SETUP.md](../../SETUP.md))
- HTTP client (Postman, Insomnia, or cURL)
- Stripe test account and API keys
- Basic understanding of REST APIs and JWT authentication

## Tutorial Steps

### Step 1: User Registration & Authentication

#### 1.1 Register a New User

First, create a user account:

**Request:**
```http
POST http://localhost:8081/api/auth/register
Content-Type: application/json

{
  "email": "alice@example.com",
  "password": "SecurePass123!",
  "firstName": "Alice",
  "lastName": "Johnson",
  "role": "User"
}
```

**Response:**
```json
{
  "userId": 42,
  "email": "alice@example.com",
  "role": "User"
}
```

**Status Code:** `201 Created`

#### 1.2 Login to Get Access Token

Authenticate to receive JWT tokens:

**Request:**
```http
POST http://localhost:8081/api/auth/login
Content-Type: application/json

{
  "email": "alice@example.com",
  "password": "SecurePass123!"
}
```

**Response:**
```json
{
  "userId": 42,
  "email": "alice@example.com",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0MiIsImVtYWlsIjoiYWxpY2VAZXhhbXBsZS5jb20iLCJyb2xlIjoiVXNlciIsImV4cCI6MTcwMTIzNDU2NywiaWF0IjoxNzAxMjMxMzY3fQ.xyz",
  "refreshToken": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6",
  "expiresIn": 3600
}
```

**Status Code:** `200 OK`

**💡 Important:** Save the `accessToken` for subsequent requests. It expires in 60 minutes.

---

### Step 2: Search for Hotels

#### 2.1 Search Hotels by Location and Dates

Search for available hotels in Riga, Latvia:

**Request:**
```http
GET http://localhost:8082/api/hotels/search?country=Latvia&city=Riga&checkIn=2024-12-20&checkOut=2024-12-25&guestsCount=2&petsAllowed=false
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
[
  {
    "hotelId": 15,
    "hotelName": "Grand Plaza Hotel",
    "country": "Latvia",
    "city": "Riga",
    "district": "Old Town",
    "petsAllowed": true,
    "minPricePerNight": 89.00,
    "availableRooms": 3
  },
  {
    "hotelId": 18,
    "hotelName": "Riverside Hotel",
    "country": "Latvia",
    "city": "Riga",
    "district": "Center",
    "petsAllowed": false,
    "minPricePerNight": 65.00,
    "availableRooms": 5
  }
]
```

**Status Code:** `200 OK`

#### 2.2 Get Detailed Hotel Information

Retrieve full details for a specific hotel:

**Request:**
```http
GET http://localhost:8082/api/hotels/15
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
{
  "id": 15,
  "ownerId": 103,
  "name": "Grand Plaza Hotel",
  "description": "Luxury hotel in the heart of Riga with modern amenities and excellent service",
  "country": "Latvia",
  "city": "Riga",
  "district": "Old Town",
  "addressLine": "Elizabetes iela 55",
  "petsAllowed": true,
  "isPetHotel": false,
  "cancelFreeDaysBefore": 7,
  "approval": "Approved",
  "submittedAt": "2024-01-15T10:30:00Z",
  "reviewedAt": "2024-01-16T14:20:00Z"
}
```

**Status Code:** `200 OK`

**💡 Note:** `cancelFreeDaysBefore: 7` means free cancellation up to 7 days before check-in.

#### 2.3 Get Room Details

Get information about a specific room:

**Request:**
```http
GET http://localhost:8082/api/rooms/55
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
{
  "id": 55,
  "hotelId": 15,
  "roomNumber": "203",
  "description": "Cozy room with city view, queen bed, and modern bathroom",
  "capacity": 2,
  "bedrooms": 1,
  "pricePerNight": 89.00,
  "visible": true,
  "petsAllowed": true,
  "accommodation": "HotelRoom",
  "createdAt": "2024-01-15T11:00:00Z"
}
```

**Status Code:** `200 OK`

---

### Step 3: Create a Reservation

#### 3.1 Calculate Total Cost

Before creating the reservation, calculate the total:

```
Check-in: 2024-12-20
Check-out: 2024-12-25
Nights: 5
Price per night: €89.00
Total: 5 × €89.00 = €445.00
```

#### 3.2 Create the Reservation

**Request:**
```http
POST http://localhost:8083/api/reservations
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "userId": 42,
  "roomId": 55,
  "startDate": "2024-12-20",
  "endDate": "2024-12-25",
  "guestsCount": 2,
  "guestsNames": "Alice Johnson, Bob Smith"
}
```

**Response:**
```json
{
  "reservationId": 150,
  "status": "Pending"
}
```

**Status Code:** `201 Created`

**💡 Important:** The reservation is created with `Pending` status. It must be paid to become `Confirmed`.

#### 3.3 Verify Reservation Details

Get the full reservation details:

**Request:**
```http
GET http://localhost:8083/api/reservations/150
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
{
  "id": 150,
  "userId": 42,
  "roomId": 55,
  "startDate": "2024-12-20",
  "endDate": "2024-12-25",
  "guestsCount": 2,
  "guestsNames": "Alice Johnson, Bob Smith",
  "status": "Pending",
  "cancellationStatus": "None",
  "createdAt": "2024-11-15T10:30:00Z",
  "cancellationRequestedAt": null,
  "cancellationReason": null
}
```

**Status Code:** `200 OK`

---

### Step 4: Process Payment

#### 4.1 Create Payment Intent

Create a Stripe payment intent for the reservation:

**Request:**
```http
POST http://localhost:8084/api/payments/create-intent
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "reservationId": 150,
  "amount": 445.00,
  "currency": "EUR"
}
```

**Response:**
```json
{
  "paymentIntentId": "pi_3ABCdefGHI123456",
  "clientSecret": "pi_3ABCdefGHI123456_secret_XYZabc789def",
  "paymentId": 88
}
```

**Status Code:** `201 Created`

**💡 Important:** The `clientSecret` is used on the frontend to confirm the payment with Stripe.js.

#### 4.2 Confirm Payment (Frontend Integration)

**Note:** This step is typically done on the frontend using Stripe.js. Here's an example:

```javascript
// Frontend code (React example)
import { loadStripe } from '@stripe/stripe-js';
import { CardElement, useStripe, useElements } from '@stripe/react-stripe-js';

async function handlePayment(clientSecret) {
  const stripe = await loadStripe('pk_test_...');
  const elements = stripe.elements();
  const cardElement = elements.create('card');
  
  const { error, paymentIntent } = await stripe.confirmCardPayment(clientSecret, {
    payment_method: {
      card: cardElement,
      billing_details: {
        name: 'Alice Johnson',
        email: 'alice@example.com'
      }
    }
  });
  
  if (error) {
    console.error('Payment failed:', error.message);
    return { success: false, error: error.message };
  }
  
  if (paymentIntent.status === 'succeeded') {
    console.log('Payment succeeded!');
    return { success: true, paymentIntentId: paymentIntent.id };
  }
}
```

#### 4.3 Stripe Webhook Processing

When payment succeeds, Stripe sends a webhook to the Payments Service:

**Webhook Event:**
```json
{
  "type": "payment_intent.succeeded",
  "data": {
    "object": {
      "id": "pi_3ABCdefGHI123456",
      "amount": 44500,
      "currency": "eur",
      "status": "succeeded",
      "charges": {
        "data": [{
          "id": "ch_3XYZabcDEF789012",
          "payment_method_details": {
            "type": "card"
          }
        }]
      }
    }
  }
}
```

**What Happens Next:**
1. Payments Service validates webhook signature
2. Updates payment status to `Succeeded`
3. Calls Reservations Service internal API
4. Reservation status changes to `Confirmed`

#### 4.4 Verify Payment Status

Check the payment details:

**Request:**
```http
GET http://localhost:8084/api/payments/88
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
{
  "id": 88,
  "reservationId": 150,
  "amount": 445.00,
  "currency": "EUR",
  "provider": "Stripe",
  "status": "Succeeded",
  "paymentMethodType": "card",
  "paymentIntentId": "pi_3ABCdefGHI123456",
  "providerPaymentId": "ch_3XYZabcDEF789012",
  "amountRefunded": 0.00,
  "refundedAt": null,
  "paidAt": "2024-11-15T10:35:00Z",
  "createdAt": "2024-11-15T10:30:00Z",
  "isActive": true,
  "errorCode": null,
  "errorMessage": null
}
```

**Status Code:** `200 OK`

---

### Step 5: Confirm Reservation Status

#### 5.1 Check Reservation is Confirmed

Verify that the reservation status changed to `Confirmed`:

**Request:**
```http
GET http://localhost:8083/api/reservations/150
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
{
  "id": 150,
  "userId": 42,
  "roomId": 55,
  "startDate": "2024-12-20",
  "endDate": "2024-12-25",
  "guestsCount": 2,
  "guestsNames": "Alice Johnson, Bob Smith",
  "status": "Confirmed",
  "cancellationStatus": "None",
  "createdAt": "2024-11-15T10:30:00Z",
  "cancellationRequestedAt": null,
  "cancellationReason": null
}
```

**Status Code:** `200 OK`

**✅ Success!** The booking is now confirmed.

---

### Step 6: View My Reservations

#### 6.1 Get All User Reservations

**Request:**
```http
GET http://localhost:8083/api/reservations/mine
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Response:**
```json
[
  {
    "id": 150,
    "roomId": 55,
    "startDate": "2024-12-20",
    "endDate": "2024-12-25",
    "guestsCount": 2,
    "status": "Confirmed",
    "createdAt": "2024-11-15T10:30:00Z"
  }
]
```

**Status Code:** `200 OK`

---

## Bonus: Cancellation Workflow

### Scenario 1: Free Cancellation (Within Policy Window)

If you cancel **7+ days before check-in**, cancellation is automatic:

**Request:**
```http
POST http://localhost:8083/api/reservations/150/cancel
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "reason": "Change of travel plans"
}
```

**Response:**
```json
{
  "message": "Reservation canceled successfully. Refund will be processed.",
  "cancellationStatus": "AutoCanceled"
}
```

**Status Code:** `200 OK`

**What Happens:**
1. Reservation status → `Canceled`
2. CancellationStatus → `AutoCanceled`
3. Payments Service processes refund automatically
4. User receives full refund to original payment method

---

### Scenario 2: Requested Cancellation (Outside Policy Window)

If you cancel **less than 7 days before check-in**, admin approval required:

**Request:**
```http
POST http://localhost:8083/api/reservations/150/cancel
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "reason": "Emergency situation"
}
```

**Response:**
```json
{
  "message": "Cancellation request submitted for admin review",
  "cancellationStatus": "Requested"
}
```

**Status Code:** `200 OK`

**What Happens:**
1. Reservation status remains `Confirmed`
2. CancellationStatus → `Requested`
3. Admin reviews the request
4. If approved:
   - Reservation status → `Canceled`
   - CancellationStatus → `AdminApproved`
   - Refund processed

---

## Complete Flow Diagram

```
┌──────────┐
│  User    │
└────┬─────┘
     │
     │ 1. Register & Login
     ▼
┌─────────────────┐
│ Users Service   │──────► JWT Token
└─────────────────┘
     │
     │ 2. Search Hotels
     ▼
┌─────────────────┐
│ Hotels Service  │──────► Hotel & Room Info
└─────────────────┘
     │
     │ 3. Create Reservation
     ▼
┌─────────────────────┐
│Reservations Service │──────► Reservation (Pending)
└─────────────────────┘
     │
     │ 4. Create Payment Intent
     ▼
┌─────────────────┐
│Payments Service │──────► Client Secret
└────┬────────────┘
     │
     │ 5. Confirm Payment (Frontend)
     ▼
┌──────────┐
│  Stripe  │
└────┬─────┘
     │
     │ 6. Webhook: payment_intent.succeeded
     ▼
┌─────────────────┐
│Payments Service │
└────┬────────────┘
     │
     │ 7. Mark Confirmed (Internal API)
     ▼
┌─────────────────────┐
│Reservations Service │──────► Reservation (Confirmed)
└─────────────────────┘
     │
     │ 8. Verify Booking
     ▼
┌──────────┐
│  User    │──────► ✅ Booking Complete!
└──────────┘
```

---

## Error Scenarios & Troubleshooting

### Error 1: Room Not Available

**Response:**
```json
{
  "error": "Room is not available for the selected dates"
}
```

**Solution:** Choose different dates or another room.

---

### Error 2: Payment Failed

**Response:**
```json
{
  "error": "Payment failed: insufficient_funds"
}
```

**Solution:** 
- Check card balance
- Use different payment method
- Reservation remains `Pending` - can retry payment

---

### Error 3: Token Expired

**Response:**
```json
{
  "error": "Token has expired"
}
```

**Solution:** Use refresh token to get new access token:

```http
POST http://localhost:8081/api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6"
}
```

---

## Testing with Stripe Test Cards

Use these test cards for different scenarios:

| Card Number | Scenario |
|-------------|----------|
| `4242 4242 4242 4242` | Successful payment |
| `4000 0000 0000 9995` | Insufficient funds |
| `4000 0000 0000 0002` | Card declined |
| `4000 0025 0000 3155` | Requires authentication (3D Secure) |

**Expiration:** Any future date (e.g., 12/25)  
**CVC:** Any 3 digits (e.g., 123)  
**ZIP:** Any 5 digits (e.g., 12345)

---

## Summary

You've successfully completed a full hotel booking flow! Here's what you learned:

1. ✅ **User Registration & Authentication** - JWT tokens
2. ✅ **Hotel Search** - Location-based filtering
3. ✅ **Reservation Creation** - Booking workflow
4. ✅ **Payment Processing** - Stripe integration
5. ✅ **Reservation Confirmation** - Webhook handling
6. ✅ **Cancellation Management** - Policy enforcement

## Next Steps

- **Explore Admin Features**: [Admin Operations Guide](ADMIN_GUIDE.md)
- **Become a Hotel Owner**: [Hotel Owner Guide](HOTEL_OWNER_GUIDE.md)
- **Deploy to Production**: [Cloud Deployment Guide](../cloud/CLOUD_DEPLOYMENT.md)
- **Review Architecture**: [Microservices Architecture](../architecture/MICROSERVICES_ARCHITECTURE.md)

---

**Need Help?**
- 📧 Email: support@hotelbooking.com
- 💬 Discord: [Join Community](https://discord.gg/hotelbooking)
- 📚 Documentation: `/docs`

Happy booking! 🏨✨
