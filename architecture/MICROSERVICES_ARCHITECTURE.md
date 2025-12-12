# Microservices Architecture Documentation

## Table of Contents
1. [Domain-Driven Design](#domain-driven-design)
2. [Data Management](#data-management)
3. [Security Architecture](#security-architecture)
4. [Token Lifecycle](#token-lifecycle)
5. [Data Protection & API Security](#data-protection--api-security)


---

## Domain-Driven Design

### Bounded Contexts

The system is organized into 4 bounded contexts, each represented by a microservice:

#### 1. Identity & Access Management (Users Service)

**Entities**:
- `User`: Core identity entity
  - Properties: Id, Email, Password (hashed), FirstName, LastName, Role
  - Behaviors: Register, Login, ChangePassword, UpdateProfile

**Value Objects**:
- `Role`: Enum (User, HotelOwner, Admin)

**Domain Events**:
- UserRegistered  
- UserLoggedIn  
- PasswordChanged  

---

#### 2. Hotel Management (Hotels Service)

**Entities**:
- `Hotel`: Hotel information  
  - Properties: Id, OwnerId, Name, Country, City, Description, PetsAllowed, CancelFreeDaysBefore, ApprovalStatus  
  - Behaviors: Submit, Approve, Reject, Update
  
- `Room`: Room inventory  
  - Properties: Id, HotelId, RoomNumber, Capacity, Bedrooms, PricePerNight, Accommodation, PetsAllowed  
  - Behaviors: Create, Update, SetVisibility

**Value Objects**:
- `ApprovalStatus`: Enum (Pending, Approved, Rejected)  
- `AccommodationType`: Enum (HotelRoom, Apartment, Studio, Suite, Villa, Cabin)

**Domain Events**:
- HotelSubmitted  
- HotelApproved  
- HotelRejected  
- RoomCreated  

---

#### 3. Booking Management (Reservations Service)

**Entities**:
- `Reservation`: Booking entity  
  - Properties: Id, UserId, RoomId, StartDate, EndDate, GuestsCount, Status, CancellationStatus  
  - Behaviors: Create, Confirm, RequestCancellation, ApproveCancellation, RejectCancellation

**Value Objects**:
- `ReservationStatus`: Enum (Pending, Confirmed, Canceled)  
- `CancellationStatus`: Enum (None, Requested, AdminApproved, AdminRejected, AutoCanceled)

**Domain Events**:
- ReservationCreated  
- ReservationConfirmed  
- CancellationRequested  
- CancellationApproved  
- ReservationCanceled  

---

#### 4. Payment Processing (Payments Service)

**Entities**:
- `Payment`: Payment transaction  
  - Properties: Id, ReservationId, Amount, Currency, Status, PaymentIntentId, AmountRefunded  
  - Behaviors: Create, MarkSucceeded, MarkFailed, Refund

**Value Objects**:
- `PaymentStatus`: Enum (RequiresPayment, Succeeded, Failed, Refunded)  
- `PaymentProvider`: Enum (Stripe)  
- `PaymentMethodType`: Enum (card, bank_transfer, wallet)

**Domain Events**:
- PaymentIntentCreated  
- PaymentSucceeded  
- PaymentFailed  
- RefundProcessed  

---

## Data Management


### Database Schema

**Shared SQL Server Instance** with separate schemas (one logical database in Azure or mssql docker container):

```
SQL Server Instance
├── users schema
│ ├── Users
│ ├── RefreshTokens
│ └── DataExportRequests
├── hotels schema
│ ├── Hotels
│ └── Rooms
├── reservations schema
│ └── Reservations
└── payments schema
└── Payments
```

### Data Consistency Strategies

**1. Eventual Consistency**:
- Payment confirmation → Reservation confirmation (webhook-based)  
- Acceptable delay for status updates between Payments and Reservations  

**2. Compensating Transactions**:
- Failed payment → Reservation remains `Pending`  
- User can retry payment  
- Admin can manually cancel stale reservations  

**3. Data Duplication**:
- Reservation stores `RoomId` (reference)  
- Hotels Service provides room details on demand via internal API  
- No cross-schema foreign keys enforced between microservices (logical boundaries stay in code and API contracts)  

---

## Security Architecture

### 1. Authentication & Authorization

**JWT Token Structure**:

```json
{
  "sub": "42",
  "email": "user@example.com",
  "role": "User",
  "exp": 1701234567,
  "iat": 1701231367
}
```
## Token Lifecycle

### Access Token
- Valid for **60 minutes**

### Refresh Token
- Valid for **7 days**
- Stored in database for revocation support

### Authorization Levels
- **User**: Basic access (own data only)  
- **HotelOwner**: Manage own hotels and rooms  
- **Admin**: Full system access (approvals, user management, monitoring)

---

## Data Protection & API Security

### Password Security
- PBKDF2 using HMACSHA256 with salt
- Minimum 8 characters  

### Encryption
- Environment variables and secrets stored in **Azure Key Vault**  
- **TLS/HTTPS** enforced for API communication in production  
- Encrypted connections to Azure SQL  

### API Key Security
- Unique key per internal service (`/internal/*` endpoints)  
- Stored in environment variables / Key Vault  
- Periodic rotation supported via configuration  

### Input Validation
- DataAnnotations on DTOs and commands  
- Centralized model validation in controllers  
- EF Core parameterization to prevent SQL injection  
---


