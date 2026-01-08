# Database Schema

## Overview

| Attribute | Value |
|-----------|-------|
| **Database** | Microsoft SQL Server |
| **ORM** | Entity Framework Core 9 |
| **Approach** | Code-First with Migrations |
| **Schema Separation** | Each microservice has its own schema |

The Hotel Booking Platform uses a **database-per-service** pattern where each microservice manages its own database schema. This ensures loose coupling and independent deployability.

## Schema Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         SQL Server Instance                         │
├─────────────────┬─────────────────┬─────────────────┬───────────────┤
│  users schema   │  hotels schema  │reservations     │ payments      │
│                 │                 │schema           │ schema        │
├─────────────────┼─────────────────┼─────────────────┼───────────────┤
│ • Users         │ • Hotels        │ • Reservations  │ • Payments    │
│ • RefreshTokens │ • Rooms         │                 │ • Refunds     │
└─────────────────┴─────────────────┴─────────────────┴───────────────┘
```

---

## Users Service Schema

### Table: users.Users

Stores user account information.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| Id | UNIQUEIDENTIFIER | PK, NOT NULL, DEFAULT NEWID() | Primary key (GUID) |
| Email | NVARCHAR(255) | NOT NULL, UNIQUE | User's email address |
| PasswordHash | NVARCHAR(255) | NOT NULL | PBKDF2 hashed password |
| Role | NVARCHAR(50) | NOT NULL, DEFAULT 'User' | "User", "HotelOwner", or "Admin" |
| IsDeleted | BIT | NOT NULL, DEFAULT 0 | Soft delete flag |
| DeletedAt | DATETIMEOFFSET | NULL | Deletion timestamp |
| CreatedAt | DATETIMEOFFSET | NOT NULL, DEFAULT SYSUTCDATETIME() | Account creation timestamp |

**Indexes:**
- `UQ_Users_Email` UNIQUE on `Email`
- `IX_Users_Role` on `Role`

---

### Table: users.RefreshTokens

Stores active refresh tokens for JWT authentication.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| Id | UNIQUEIDENTIFIER | PK, NOT NULL, DEFAULT NEWID() | Primary key (GUID) |
| UserId | UNIQUEIDENTIFIER | FK → Users.Id, NOT NULL | Associated user |
| Token | NVARCHAR(500) | NOT NULL, UNIQUE | Refresh token value |
| ExpiresAt | DATETIMEOFFSET | NOT NULL | Token expiration time |
| IsRevoked | BIT | NOT NULL, DEFAULT 0 | Revocation flag |
| CreatedAt | DATETIMEOFFSET | NOT NULL, DEFAULT SYSUTCDATETIME() | Token creation time |
| RevokedAt | DATETIMEOFFSET | NULL | When token was revoked |
| ReplacedByToken | NVARCHAR(500) | NULL | Token that replaced this one |

**Indexes:**
- `UQ_RefreshTokens_Token` UNIQUE on `Token`
- `IX_RefreshTokens_UserId` on `UserId`

---

## Hotels Service Schema

### Table: hotels.Hotels

Stores hotel property information.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| Id | UNIQUEIDENTIFIER | PK, NOT NULL, DEFAULT NEWID() | Primary key (GUID) |
| OwnerId | UNIQUEIDENTIFIER | NOT NULL | Hotel owner's user ID (logical FK) |
| Name | NVARCHAR(255) | NOT NULL | Hotel name |
| Description | NVARCHAR(MAX) | NULL | Hotel description |
| Country | NVARCHAR(100) | NOT NULL | Country |
| City | NVARCHAR(120) | NOT NULL | City |
| District | NVARCHAR(120) | NULL | District/area |
| AddressLine | NVARCHAR(300) | NULL | Street address |
| PetsAllowed | BIT | NOT NULL, DEFAULT 0 | Pets allowed flag |
| IsPetHotel | BIT | NOT NULL, DEFAULT 0 | Specialized pet hotel |
| CancelFreeDaysBefore | INT | NOT NULL, DEFAULT 0, CHECK >= 0 | Free cancellation window |
| Approval | NVARCHAR(20) | NOT NULL, DEFAULT 'Pending' | Pending/Approved/Rejected |
| SubmittedAt | DATETIMEOFFSET | NOT NULL, DEFAULT SYSUTCDATETIME() | Submission timestamp |
| ReviewedAt | DATETIMEOFFSET | NULL | Admin review timestamp |

**Indexes:**
- `IX_Hotels_OwnerId` on `OwnerId`
- `IX_Hotels_Country_City` on `Country, City`
- `IX_Hotels_Approval` on `Approval`

---

### Table: hotels.Rooms

Stores individual room information for hotels.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| Id | UNIQUEIDENTIFIER | PK, NOT NULL, DEFAULT NEWID() | Primary key (GUID) |
| HotelId | UNIQUEIDENTIFIER | FK → Hotels.Id, NOT NULL | Parent hotel |
| RoomNumber | NVARCHAR(50) | NOT NULL | Room identifier |
| Description | NVARCHAR(MAX) | NULL | Room description |
| Capacity | INT | NOT NULL, CHECK > 0 | Maximum occupancy |
| Bedrooms | INT | NOT NULL, CHECK >= 0 | Number of bedrooms |
| PricePerNight | DECIMAL(18,2) | NOT NULL, CHECK > 0 | Nightly rate |
| Visible | BIT | NOT NULL, DEFAULT 1 | Availability flag |
| PetsAllowed | BIT | NOT NULL, DEFAULT 0 | Pet friendly flag |
| Accommodation | NVARCHAR(50) | NOT NULL | Enum: HotelRoom/Apartment/House/Cabin/Capsule |
| CreatedAt | DATETIMEOFFSET | NOT NULL, DEFAULT SYSUTCDATETIME() | Creation timestamp |

**Indexes:**
- `IX_Rooms_HotelId` on `HotelId`
- `UQ_Rooms_HotelId_RoomNumber` UNIQUE on `(HotelId, RoomNumber)`

---

## Reservations Service Schema

### Table: reservations.Reservations

Stores booking/reservation information.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| Id | UNIQUEIDENTIFIER | PK, NOT NULL, DEFAULT NEWID() | Primary key (GUID) |
| UserId | UNIQUEIDENTIFIER | NOT NULL | Guest's user ID (logical FK) |
| RoomId | UNIQUEIDENTIFIER | NOT NULL | Booked room ID (logical FK) |
| StartDate | DATE | NOT NULL | Check-in date |
| EndDate | DATE | NOT NULL, CHECK > StartDate | Check-out date |
| NumberOfGuests | INT | NOT NULL, CHECK > 0 | Number of guests |
| Status | NVARCHAR(30) | NOT NULL, DEFAULT 'Active' | Booking status |
| CreatedAt | DATETIMEOFFSET | NOT NULL, DEFAULT SYSUTCDATETIME() | Booking creation time |
| CancelledAt | DATETIMEOFFSET | NULL | Cancellation timestamp |
| ConfirmedAt | DATETIMEOFFSET | NULL | Confirmation timestamp |

**Status Values:**
- `Active` - Booking confirmed and active
- `Cancelled` - Cancelled by user/admin
- `RefundPending` - Awaiting refund
- `Completed` - Stay completed

**Indexes:**
- `IX_Reservations_UserId` on `UserId`
- `IX_Reservations_RoomId` on `RoomId`
- `IX_Reservations_Status` on `Status`
- `IX_Reservations_StartDate_EndDate` on `(StartDate, EndDate)`

---

## Payments Service Schema

### Table: payments.Payments

Stores payment transaction information.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| Id | UNIQUEIDENTIFIER | PK, NOT NULL, DEFAULT NEWID() | Primary key (GUID) |
| ReservationId | UNIQUEIDENTIFIER | NOT NULL, UNIQUE | Associated reservation (logical FK) |
| Amount | DECIMAL(18,2) | NOT NULL, CHECK > 0 | Payment amount |
| Currency | NVARCHAR(10) | NOT NULL, DEFAULT 'USD' | Currency code |
| Status | NVARCHAR(30) | NOT NULL, DEFAULT 'RequiresPayment' | Payment status |
| StripePaymentIntentId | NVARCHAR(500) | NULL | Stripe PI identifier |
| CreatedAt | DATETIMEOFFSET | NOT NULL, DEFAULT SYSUTCDATETIME() | Payment initiated |
| PaidAt | DATETIMEOFFSET | NULL | Payment completed |

**Status Values:**
- `RequiresPayment` - Intent created, awaiting payment
- `Processing` - Payment in progress
- `Succeeded` - Payment completed
- `Failed` - Payment failed
- `Refunded` - Payment refunded

**Indexes:**
- `UQ_Payments_ReservationId` UNIQUE on `ReservationId`
- `IX_Payments_Status` on `Status`
- `IX_Payments_StripePaymentIntentId` on `StripePaymentIntentId`

---

### Table: payments.Refunds

Stores refund transaction information.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| Id | UNIQUEIDENTIFIER | PK | Primary key (GUID) |
| PaymentId | UNIQUEIDENTIFIER | FK → Payments.Id, NOT NULL | Original payment |
| Amount | DECIMAL(18,2) | NOT NULL | Refund amount |
| Reason | NVARCHAR(500) | NULL | Refund reason |
| Status | NVARCHAR(50) | NOT NULL | Refund status |
| StripeRefundId | NVARCHAR(500) | NULL | Stripe refund identifier |
| CreatedAt | DATETIME2 | NOT NULL | Refund initiated |
| CompletedAt | DATETIME2 | NULL | Refund completed |

**Indexes:**
- `IX_Refunds_PaymentId` on `PaymentId`

---

## Entity Relationship Diagram

```
USERS SERVICE                    HOTELS SERVICE
┌─────────────────┐              ┌─────────────────┐
│     Users       │              │     Hotels      │
├─────────────────┤              ├─────────────────┤
│ Id (PK)         │              │ Id (PK)         │
│ Email           │              │ Name            │
│ PasswordHash    │              │ OwnerId ────────┼──▶ Users.Id
│ Role            │              │ Country         │
│ IsDeleted       │              │ City            │
│ DeletedAt       │              │ Approval        │
│ CreatedAt       │              └────────┬────────┘
└────────┬────────┘                       │ 1:N
         │ 1:N                            ▼
         ▼                       ┌─────────────────┐
┌─────────────────┐              │     Rooms       │
│  RefreshTokens  │              ├─────────────────┤
├─────────────────┤              │ Id (PK)         │
│ Id (PK)         │              │ HotelId (FK)    │
│ UserId (FK)     │              │ RoomNumber      │
│ Token           │              │ Capacity        │
│ ExpiresAt       │              │ PricePerNight   │
│ IsRevoked       │              │ Accommodation   │
└─────────────────┘              └─────────────────┘

RESERVATIONS SERVICE             PAYMENTS SERVICE
┌─────────────────┐              ┌─────────────────┐
│  Reservations   │              │    Payments     │
├─────────────────┤              ├─────────────────┤
│ Id (PK)         │◀────────────▶│ Id (PK)         │
│ UserId ─────────┼──▶Users.Id   │ ReservationId   │
│ RoomId ─────────┼──▶Rooms.Id   │ Amount          │
│ StartDate       │              │ Currency        │
│ EndDate         │              │ Status          │
│ NumberOfGuests  │              │ StripeIntentId  │
│ Status          │              │ CreatedAt       │
│ CreatedAt       │              │ PaidAt          │
└─────────────────┘              └─────────────────┘
```

**Note:** Cross-service references are by ID only, not foreign keys. Each service maintains data consistency through API calls, not database constraints.

---

## Migrations

Entity Framework Core manages schema migrations for each service.

### Users Service Migrations

| Migration | Description | Date |
|-----------|-------------|------|
| InitialCreate | Users and RefreshTokens tables | 2025-11-15 |
| AddUserRole | Added Role column | 2025-11-20 |
| AddRefreshTokenRevocation | Added RevokedAt column | 2025-12-01 |

### Hotels Service Migrations

| Migration | Description | Date |
|-----------|-------------|------|
| InitialCreate | Hotels and Rooms tables | 2025-11-15 |
| AddHotelApproval | Added IsApproved flag | 2025-11-22 |
| AddCancellationPolicy | Added CancelFreeDaysBefore | 2025-12-05 |

### Reservations Service Migrations

| Migration | Description | Date |
|-----------|-------------|------|
| InitialCreate | Reservations table | 2025-11-18 |
| AddCancellationFields | Added cancellation tracking | 2025-12-10 |
| AddPaymentReference | Added PaymentId | 2025-12-15 |

### Payments Service Migrations

| Migration | Description | Date |
|-----------|-------------|------|
| InitialCreate | Payments table | 2025-11-20 |
| AddStripeFields | Added Stripe integration fields | 2025-12-01 |
| AddRefunds | Added Refunds table | 2025-12-20 |

---

## Running Migrations

Apply migrations for each service:

```bash
# Users Service
cd hotel-booking-users-service/Api
dotnet ef database update --project ../Infrastructure

# Hotels Service
cd hotel-booking-hotels-service/Api
dotnet ef database update --project ../Infrastructure

# Reservations Service
cd hotel-booking-reservations-service/Api
dotnet ef database update --project ../Infrastructure

# Payments Service
cd hotel-booking-payments-service/Api
dotnet ef database update --project ../Infrastructure
```

---

## Test Data Seeding

Each service can seed test data via its `DbContext`:

```csharp
// In Program.cs or migration
if (app.Environment.IsDevelopment())
{
    using var scope = app.Services.CreateScope();
    var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    await context.SeedTestDataAsync();
}
```
