# System Architecture Diagrams

This document provides visual representations of the Hotel Booking microservices architecture.

## Table of Contents
1. [System Overview](#system-overview)
2. [Service Architecture](#service-architecture)
3. [Data Flow Diagrams](#data-flow-diagrams)
4. [Deployment Architecture](#deployment-architecture)
5. [Security Architecture](#security-architecture)

---

## System Overview

### High-Level Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                          Client Applications                            │
│                  (Web Browser)								          │
└───────┬────────────┬────────────┬────────────┬──────────────────────────┘
        │            │            │            │   HTTPS/REST
        ▼            ▼            ▼            ▼
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Users   │  │  Hotels  │  │Reserv-   │  │ Payments │
│ Service  │  │ Service  │  │ations    │  │ Service  │
│  :8081   │  │  :8082   │  │ Service  │  │  :8084   │
│          │  │          │  │  :8083   │  │          │
└────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │             │
     │    ┌────────┴─────────────┴─────────────┘
     │    │
     │    │    Service-to-Service Communication
     │    │    (REST + API Keys)
     │    │
     ▼    ▼
  ┌───────────────────────────────────┐
  │   SQL Server: HotelBooking DB     │
  │  ┌──────┬────────┬────────┬──────┐│
  │  │users │hotels  │reserv. │paym  ││
  │  │schema│schema  │schema  │schema││
  │  └──────┴────────┴────────┴──────┘│
  └───────────────────────────────────┘

External Services:
┌──────────────┐
│  Stripe API  │ ◄────────────── Payments Service
└──────────────┘
```

---

## Service Architecture

### Users Service

```
┌─────────────────────────────────────────────────────────┐
│                    Users Service :8081                   │
├─────────────────────────────────────────────────────────┤
│  API Layer (Controllers)                                │
│  ├─ AuthController                                      │
│  │   ├─ POST /api/auth/register                        │
│  │   ├─ POST /api/auth/login                           │
│  │   └─ POST /api/auth/refresh                         │
│  └─ UsersController                                     │
│      ├─ GET /api/users                                  │
│      ├─ GET /api/users/{id}                            │
│      ├─ PATCH /api/users/{id}                          │
│      └─ PATCH /api/users/{id}/password                 │
├─────────────────────────────────────────────────────────┤
│  Application Layer (Handlers)                           │
│  ├─ RegisterUserHandler                                 │
│  ├─ LoginHandler                                        │
│  ├─ RefreshTokenHandler                                 │
│  ├─ UpdateUserHandler                                   │
│  └─ ChangePasswordHandler                               │
├─────────────────────────────────────────────────────────┤
│  Domain Layer                                           │
│  └─ User (Entity)                                       │
│      ├─ Id, Email, PasswordHash                        │
│      ├─ Role, IsDeleted, DeletedAt                     │
│      └─ Methods: Register(), ChangePassword()          │
├─────────────────────────────────────────────────────────┤
│  Infrastructure Layer                                   │
│  ├─ UsersRepository                                     │
│  ├─ RefreshTokenRepository                              │
│  ├─ JwtTokenGenerator                                   │
│  └─ PasswordHasher (PBKDF2)                            │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
         ┌──────────────┐
         │   users.Users│
         └──────────────┘
```

### Hotels Service

```
┌─────────────────────────────────────────────────────────┐
│                   Hotels Service :8082                   │
├─────────────────────────────────────────────────────────┤
│  API Layer                                              │
│  ├─ HotelsController                                    │
│  │   ├─ POST /api/hotels                               │
│  │   ├─ GET /api/hotels/{id}                          │
│  │   ├─ PATCH /api/hotels/{id}                        │
│  │   ├─ GET /api/hotels/mine                          │
│  │   └─ GET /api/hotels/search                        │
│  ├─ RoomsController                                     │
│  │   ├─ POST /api/rooms                                │
│  │   ├─ GET /api/rooms/{id}                           │
│  │   └─ PATCH /api/rooms/{id}                         │
│  └─ AdminController                                     │
│      ├─ GET /api/admin/hotels/pending                  │
│      ├─ POST /api/admin/hotels/{id}/approve           │
│      └─ POST /api/admin/hotels/{id}/reject            │
├─────────────────────────────────────────────────────────┤
│  Application Layer                                      │
│  ├─ CreateHotelHandler                                  │
│  ├─ UpdateHotelHandler                                  │
│  ├─ SearchHotelsHandler                                 │
│  ├─ CreateRoomHandler                                   │
│  └─ ApproveHotelHandler                                 │
├─────────────────────────────────────────────────────────┤
│  Domain Layer                                           │
│  ├─ Hotel (Aggregate Root)                             │
│  │   ├─ Id, OwnerId, Name, Country, City              │
│  │   ├─ ApprovalStatus (Pending/Approved/Rejected)    │
│  │   └─ CancelFreeDaysBefore                          │
│  └─ Room (Entity)                                       │
│      ├─ Id, HotelId, RoomNumber                        │
│      ├─ Capacity, Bedrooms, PricePerNight             │
│      └─ Visible, PetsAllowed, Accommodation           │
├─────────────────────────────────────────────────────────┤
│  Infrastructure Layer                                   │
│  ├─ HotelsRepository                                    │
│  ├─ RoomsRepository                                     │
│  └─ AvailabilityChecker                                 │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
         ┌──────────────┐
         │ hotels.Hotels│
         └──────────────┘
```

### Reservations Service

```
┌─────────────────────────────────────────────────────────┐
│              Reservations Service :8083                  │
├─────────────────────────────────────────────────────────┤
│  API Layer                                              │
│  ├─ ReservationsController                              │
│  │   ├─ POST /api/reservations                         │
│  │   ├─ GET /api/reservations/{id}                    │
│  │   ├─ GET /api/reservations/mine                    │
│  │   └─ POST /api/reservations/{id}/cancel           │
│  └─ AdminReservationsController                         │
│      ├─ GET /api/admin/reservations                    │
│      ├─ POST /api/admin/reservations/{id}/approve     │
│      └─ POST /api/admin/reservations/{id}/reject      │
├─────────────────────────────────────────────────────────┤
│  Application Layer                                      │
│  ├─ CreateReservationHandler                            │
│  │   └─ Calls Hotels Service (room availability)       │
│  ├─ CancelReservationHandler                            │
│  │   └─ Smart cancellation logic                       │
│  ├─ ApproveCancellationHandler                          │
│  │   └─ Calls Payments Service (refund)               │
│  └─ RejectCancellationHandler                           │
├─────────────────────────────────────────────────────────┤
│  Domain Layer                                           │
│  └─ Reservation (Aggregate Root)                       │
│      ├─ Id, UserId, RoomId                             │
│      ├─ StartDate, EndDate, GuestsCount               │
│      ├─ Status (Pending/Confirmed/Canceled)           │
│      ├─ CancellationStatus                            │
│      └─ Methods: Confirm(), Cancel(), Request...()    │
├─────────────────────────────────────────────────────────┤
│  Infrastructure Layer                                   │
│  ├─ ReservationsRepository                              │
│  ├─ HotelsServiceClient (HTTP)                         │
│  └─ CancellationPolicyEvaluator                        │
└────────────────┬────────────────────────────────────────┘
                 │
                 ▼
         ┌──────────────────────┐
         │reservations.         │
         │    Reservations      │
         └──────────────────────┘
```

### Payments Service

```
┌─────────────────────────────────────────────────────────┐
│                 Payments Service :8084                   │
├─────────────────────────────────────────────────────────┤
│  API Layer                                              │
│  ├─ PaymentsController                                  │
│  │   ├─ POST /api/payments/create-intent              │
│  │   ├─ POST /api/payments/confirm                    │
│  │   ├─ POST /api/payments/refund                     │
│  │   ├─ GET /api/payments/{id}                        │
│  │   └─ GET /api/payments/reservation/{id}           │
│  └─ StripeWebhookController                             │
│      └─ POST /api/webhooks/stripe                      │
├─────────────────────────────────────────────────────────┤
│  Application Layer                                      │
│  ├─ CreatePaymentIntentHandler                          │
│  │   └─ Calls Stripe API                               │
│  ├─ ConfirmPaymentHandler                               │
│  ├─ RefundPaymentHandler                                │
│  │   └─ Calls Stripe API                               │
│  └─ StripeWebhookHandler                                │
│      └─ Calls Reservations Service                     │
├─────────────────────────────────────────────────────────┤
│  Domain Layer                                           │
│  └─ Payment (Aggregate Root)                           │
│      ├─ Id, ReservationId, Amount, Currency            │
│      ├─ Status (RequiresPayment/Succeeded/...)        │
│      ├─ PaymentIntentId, ProviderPaymentId            │
│      ├─ AmountRefunded                                 │
│      └─ Methods: MarkSucceeded(), Refund()            │
├─────────────────────────────────────────────────────────┤
│  Infrastructure Layer                                   │
│  ├─ PaymentsRepository                                  │
│  ├─ StripePaymentService                                │
│  ├─ StripeWebhookValidator                              │
│  └─ ReservationsServiceClient (HTTP)                   │
└────────────────┬───────────────────┬────────────────────┘
                 │                   │
                 ▼                   ▼
         ┌──────────────┐    ┌──────────────┐
         │ payments.    │    │  Stripe API  │
         │  Payments    │    └──────────────┘
         └──────────────┘
```

---

## Data Flow Diagrams

### Complete Booking Flow

```
┌──────┐                                                              
│Client│                                                              
└──┬───┘                                                              
   │                                                                  
   │ 1. Register/Login                                               
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                    ┌───────────────────┐        
   │                                    │  Users Service    │        
   │                                    │  - Validate creds │        
   │                                    │  - Generate JWT   │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │ ◄────────────────────────────────────────────┘                  
   │ JWT Token                                                        
   │                                                                  
   │ 2. Search Hotels (with JWT)                                     
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                    ┌───────────────────┐        
   │                                    │  Hotels Service   │        
   │                                    │  - Query approved │        
   │                                    │    hotels         │        
   │                                    │  - Check avail.   │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │ ◄────────────────────────────────────────────┘                  
   │ Hotel Results                                                    
   │                                                                  
   │ 3. Create Reservation                                           
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                    ┌───────────────────┐        
   │                                    │ Reservations Svc  │        
   │                                    │  - Validate avail │        
   │                                    │  - Create booking │        
   │                                    │  Status: Pending  │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │ ◄────────────────────────────────────────────┘                  
   │ Reservation ID (Pending)                                        
   │                                                                  
   │ 4. Create Payment Intent                                        
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                    ┌───────────────────┐        
   │                                    │  Payments Service │        
   │                                    │  - Create intent  │        
   │                                    │    in Stripe      │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │ ◄────────────────────────────────────────────┘                  
   │ Client Secret                                                    
   │                                                                  
   │ 5. Confirm Payment (Stripe.js)                                  
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                         ┌─────────┐             
   │                                         │ Stripe  │             
   │                                         │   API   │             
   │                                         └────┬────┘             
   │                                              │                  
   │                    6. Webhook: payment_intent.succeeded          
   │                                              │                  
   │                                              ▼                  
   │                                    ┌───────────────────┐        
   │                                    │  Payments Service │        
   │                                    │  - Validate       │        
   │                                    │  - Update status  │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │                          7. Mark Confirmed (Internal API)        
   │                                              │                  
   │                                              ▼                  
   │                                    ┌───────────────────┐        
   │                                    │ Reservations Svc  │        
   │                                    │  Status:Confirmed │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │ 8. Poll/Check Status                        │                  
   │ ◄────────────────────────────────────────────┘                  
   │ Booking Confirmed ✅                                            
   │                                                                  
```

### Cancellation Flow (Outside Free Period)

```
┌──────┐                                                              
│Client│                                                              
└──┬───┘                                                              
   │                                                                  
   │ 1. Request Cancellation                                         
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                    ┌───────────────────┐        
   │                                    │ Reservations Svc  │        
   │                                    │  - Check policy   │        
   │                                    │  - Outside free   │        
   │                                    │  - Status:        │        
   │                                    │    Requested      │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │ ◄────────────────────────────────────────────┘                  
   │ "Request submitted for review"                                  
   │                                                                  
   │                                                                  
┌──────┐                                                              
│Admin │                                                              
└──┬───┘                                                              
   │                                                                  
   │ 2. Review Request                                               
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                    ┌───────────────────┐        
   │                                    │ Reservations Svc  │        
   │                                    │  - Get pending    │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │ ◄────────────────────────────────────────────┘                  
   │ Pending Cancellations                                           
   │                                                                  
   │ 3. Approve Cancellation                                         
   ├──────────────────────────────────────────────┐                  
   │                                               ▼                  
   │                                    ┌───────────────────┐        
   │                                    │ Reservations Svc  │        
   │                                    │  - Update status  │        
   │                                    │  Status:Canceled  │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │                    4. Process Refund (Internal API)              
   │                                              │                  
   │                                              ▼                  
   │                                    ┌───────────────────┐        
   │                                    │  Payments Service │        
   │                                    │  - Refund via     │        
   │                                    │    Stripe         │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │                                              ▼                  
   │                                         ┌─────────┐             
   │                                         │ Stripe  │             
   │                                         │   API   │             
   │                                         └────┬────┘             
   │                                              │                  
   │                          5. Webhook: refund.succeeded            
   │                                              │                  
   │                                              ▼                  
   │                                    ┌───────────────────┐        
   │                                    │  Payments Service │        
   │                                    │  - Mark refunded  │        
   │                                    └─────────┬─────────┘        
   │                                              │                  
   │              6. Notify Completion (Internal API)                 
   │                                              │                  
   │                                              ▼                  
   │                                    ┌───────────────────┐        
   │                                    │ Reservations Svc  │        
   │                                    │  - Finalize       │        
   │                                    └───────────────────┘        
   │                                                                  
```

---

## Deployment Architecture

### Docker Compose (Local Development)

```
┌──────────────────────────────────────────────────────────────┐
│                      Docker Host                             │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  Docker Network: hotel-booking-network                 │  │
│  │                                                        │  │
│  │  ┌──────────────┐  ┌──────────────┐                    │  │
│  │  │  users-svc   │  │  hotels-svc  │                    │  │
│  │  │   :8081      │  │   :8082      │                    │  │
│  │  └──────┬───────┘  └──────┬───────┘                    │  │
│  │         │                  │                           │  │
│  │  ┌──────┴───────┬──────────┴───────┐                   │  │
│  │  │ reserv-svc   │  payments-svc    │                   │  │
│  │  │   :8083      │    :8084         │                   │  │
│  │  └──────┬───────┴──────────┬───────┘                   │  │
│  │         │                  │                           │  │
│  │         └──────────┬───────┘                           │  │
│  │                    │                                   │  │
│  │         ┌──────────┴───────────┐                       │  │
│  │         │   SQL Server :1433   │                       │  │
│  │         └──────────────────────┘                       │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Volumes:                                                    │
│  - sqlserver-data (persistent database storage)              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```



## Security Architecture

### Authentication Flow

```
┌────────┐
│ Client │
└───┬────┘
    │
    │ 1. POST /api/auth/login
    │    { email, password }
    ▼
┌────────────────┐
│ Users Service  │
└───┬────────────┘
    │
    │ 2. Validate credentials
    │    - Hash comparison (BCrypt)
    │    - Check user exists
    ▼
┌─────────────────┐
│ Password Valid? │
└───┬─────────────┘
    │ Yes
    ▼
┌────────────────┐
│ Generate Tokens│
│ - Access Token │  JWT (60 min)
│   {              │
│     sub: userId  │
│     email: email │
│     role: role   │
│     exp: 3600    │
│   }              │
│ - Refresh Token │  UUID (7 days)
│   Stored in DB   │
└───┬────────────┘
    │
    │ 3. Return tokens
    ▼
┌────────┐
│ Client │
│ Stores:│
│ - Access Token  │ (Memory/Cookie)
│ - Refresh Token │ (HttpOnly Cookie)
└───┬────────────┘
    │
    │ 4. API Request
    │    Authorization: Bearer <access-token>
    ▼
┌────────────────┐
│ Any Service    │
│ - Validate JWT │
│ - Check exp    │
│ - Extract role │
└───┬────────────┘
    │
    │ 5. Authorized request
    ▼
┌────────────────┐
│ Business Logic │
└────────────────┘
```

### Service-to-Service Authentication

```
┌──────────────────┐
│Reservations Svc  │
└────────┬─────────┘
         │
         │ Internal API Call
         │ X-API-Key: <hotels-api-key>
         │
         │ GET /internal/rooms/55/details
         ▼
┌──────────────────┐
│  Hotels Service  │
│                  │
│  Middleware:     │
│  -Check X-API-Key│
│  - Validate key  │
│  - Allow if valid│
└────────┬─────────┘
         │
         │ Return room details
         ▼
┌──────────────────┐
│Reservations Svc  │
└──────────────────┘
```

---

## Database Schema

### Simplified ER Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         HotelBooking (SQL Database)                          │
├──────────────────────────────────────────────────────────────────────────────┤
│  Schema: users                                                               │
│  ──────────────────────────────────────────────────────────────────────────  │
│  Users                                                                       │
│  ├─ Id (PK)                                                                  │
│  ├─ Email                                                                    │
│  ├─ PasswordHash                                                             │
│  ├─ Role (User / HotelOwner / Admin)                                         │
│  ├─ IsDeleted                                                                │
│  ├─ DeletedAt                                                                │
│  └─ CreatedAt                                                                │
│                                                                              │
│  RefreshTokens                                                               │
│  ├─ Id (PK)                                                                  │
│  ├─ UserId (FK → users.Users.Id)                                             │
│  ├─ Token                                                                    │
│  ├─ ExpiresAt                                                                │
│  ├─ CreatedAt                                                                │
│  ├─ IsRevoked                                                                │
│  ├─ RevokedAt                                                                │
│  └─ ReplacedByToken                                                          │
│                                                                              │
│  DataExportRequests                                                          │
│  ├─ Id (PK)                                                                  │
│  ├─ UserId (FK → users.Users.Id)                                             │
│  ├─ Status                                                                   │
│  ├─ RequestedAt                                                              │
│  ├─ CompletedAt                                                              │
│  ├─ ExpiresAt                                                                │
│  ├─ ExportFilePath                                                           │
│  └─ ErrorMessage                                                             │
│                                                                              │
│  --------------------------------------------------------------------------  │
│  Schema: hotels                                                              │
│  ──────────────────────────────────────────────────────────────────────────  │
│  Hotels                                                                      │
│  ├─ Id (PK)                                                                  │
│  ├─ OwnerId (reference to users.Users.Id)                                    │
│  ├─ Name                                                                     │
│  ├─ Description                                                              │
│  ├─ MainImageUrl                                                             │
│  ├─ Country                                                                  │
│  ├─ City                                                                     │
│  ├─ District                                                                 │
│  ├─ AddressLine                                                              │
│  ├─ PetsAllowed                                                              │
│  ├─ IsPetHotel                                                               │
│  ├─ CancelFreeDaysBefore                                                     │
│  ├─ Approval (Pending / Approved / Rejected)                                 │
│  ├─ SubmittedAt                                                              │
│  └─ ReviewedAt                                                               │
│                    │                                                         │
│                    │ 1 : N                                                   │
│                    ▼                                                         │
│  Rooms                                                                       │
│  ├─ Id (PK)                                                                  │
│  ├─ HotelId (FK → hotels.Hotels.Id)                                          │
│  ├─ RoomNumber                                                               │
│  ├─ Description                                                              │
│  ├─ Capacity                                                                 │
│  ├─ Bedrooms                                                                 │
│  ├─ PricePerNight                                                            │
│  ├─ MainImageUrl                                                             │
│  ├─ Visible                                                                  │
│  ├─ PetsAllowed                                                              │
│  ├─ Accommodation (enum: HotelRoom / Apartment / House / Cabin / Capsule)    │
│  └─ CreatedAt                                                                │
│                                                                              │
│  --------------------------------------------------------------------------  │
│  Schema: reservations                                                        │
│  ──────────────────────────────────────────────────────────────────────────  │
│  Reservations                                                                │
│  ├─ Id (PK)                                                                  │
│  ├─ UserId (reference to users.Users.Id)                                     │
│  ├─ RoomId (reference to hotels.Rooms.Id)                                    │
│  ├─ StartDate                                                                │
│  ├─ EndDate                                                                  │
│  ├─ GuestsCount                                                              │
│  ├─ GuestsNames (JSON array of names)                                        │
│  ├─ Status (Pending / Held / Confirmed / Canceled)                           │
│  ├─ CreatedAt                                                                │
│  ├─ CancellationRequestedAt                                                  │
│  ├─ CancellationReason                                                       │
│  └─ CancellationStatus (None / Requested / AutoCanceled /                    │
│                          AdminApproved / AdminRejected)                      │
│                                                                              │
│  --------------------------------------------------------------------------  │
│  Schema: payments                                                            │
│  ──────────────────────────────────────────────────────────────────────────  │
│  Payments                                                                    │
│  ├─ Id (PK)                                                                  │
│  ├─ ReservationId (reference to reservations.Reservations.Id)               │
│  ├─ Amount                                                                   │
│  ├─ Currency                                                                 │
│  ├─ Provider (Stripe)                                                        │
│  ├─ Status (RequiresPayment / RequiresAction / Processing /                  │
│             Succeeded / Failed / Refunded / Canceled)                        │
│  ├─ PaymentMethodType (Card / BankTransfer / Wallet)                         │
│  ├─ PaymentIntentId                                                          │
│  ├─ ProviderPaymentId                                                        │
│  ├─ AmountRefunded                                                           │
│  ├─ RefundedAt                                                               │
│  ├─ PaidAt                                                                   │
│  ├─ CreatedAt                                                                │
│  ├─ IsActive                                                                 │
│  ├─ LastProviderEventId                                                      │
│  ├─ ErrorCode                                                                │
│  └─ ErrorMessage                                                             │
└──────────────────────────────────────────────────────────────────────────────┘

```

