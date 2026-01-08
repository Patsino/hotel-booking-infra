# 3. User Guide

This section provides instructions for end users on how to use the application via Swagger UI.

## Contents

- [Features Walkthrough](features.md)
- [FAQ & Troubleshooting](faq.md)

## Getting Started

### System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **Browser** | Chrome 90+, Firefox 88+, Safari 14+, Edge 90+ | Latest version |
| **Screen Resolution** | 1280x720 | 1920x1080 |
| **Internet** | Required | Stable connection |
| **Device** | Desktop | - |

### Accessing the Application

The Hotel Booking Platform is accessed via Swagger UI for API interaction:

1. Open your web browser
2. Navigate to one of the service Swagger URLs:
   - **Users Service**: https://hotel-booking-users-api-csbghtd2f9cph7g5.northeurope-01.azurewebsites.net/swagger/index.html
   - **Hotels Service**: https://hotel-booking-hotels-api-evhhefafhhbrgrbs.northeurope-01.azurewebsites.net/swagger/index.html
   - **Reservations Service**: https://hotel-booking-reservations-api-dwfzh9bydth3fke0.northeurope-01.azurewebsites.net/swagger/index.html
   - **Payments Service**: https://hotel-booking-payments-api-gch8e7fyeqenfje8.northeurope-01.azurewebsites.net/swagger/index.html

### First Launch

#### Step 1: Register a User Account

1. Go to Users Service Swagger UI
2. Expand `POST /api/auth/register`
3. Click "Try it out"
4. Enter registration details:
   ```json
   {
     "email": "your-email@example.com",
     "password": "YourPassword123!",
   }
   ```
5. Click "Execute"
6. You should receive a 201 Created response

#### Step 2: Login to Get Token

1. Expand `POST /api/auth/login`
2. Click "Try it out"
3. Enter your credentials:
   ```json
   {
     "email": "your-email@example.com",
     "password": "YourPassword123!"
   }
   ```
4. Click "Execute"
5. Copy the `accessToken` from the response

#### Step 3: Authorize Swagger UI

1. Click the "Authorize" button (lock icon) at the top
2. Enter: `Bearer <your-access-token>`
3. Click "Authorize"
4. Now all authenticated endpoints will include your token

## Quick Start Guide

| Task | Service | Endpoint |
|------|---------|----------|
| Register account | Users | `POST /api/auth/register` |
| Login | Users | `POST /api/auth/login` |
| Search hotels | Hotels | `GET /api/hotels/search` |
| View hotel details | Hotels | `GET /api/hotels/{id}` |
| Create reservation | Reservations | `POST /api/reservations` |
| Pay for booking | Payments | `POST /api/payments/create-intent` |
| View my bookings | Reservations | `GET /api/reservations/mine` |
| Cancel booking | Reservations | `POST /api/reservations/{id}/cancel` |

## User Roles

| Role | Permissions | Access Level |
|------|-------------|--------------|
| **User** | Search hotels, create reservations, make payments, view own bookings | Basic user access |
| **HotelOwner** | All User permissions + submit hotels, manage rooms, view hotel bookings | Property management |
| **Admin** | All permissions + approve hotels, manage cancellation requests, view all data | Full system access |

## Test Accounts (Seeded Data)

The system is seeded with test accounts:

| Email | Password | Role |
|-------|----------|------|
| admin@example.com | (configured in system) | Admin |
| owner@example.com | (configured in system) | HotelOwner |
| user@example.com | (configured in system) | User |

Contact your administrator for the seeded account passwords.
