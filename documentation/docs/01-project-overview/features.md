# Features & Requirements

## Epics Overview

| Epic | Description | Stories | Status |
|------|-------------|---------|--------|
| E1: User Management | Authentication, authorization, and user profile management | 8 | ✅ |
| E2: Hotel Management | Hotel and room CRUD operations with approval workflow | 11 | ✅ |
| E3: Reservation Management | Booking lifecycle with smart cancellation policies | 7 | ✅ |
| E4: Payment Processing | Stripe integration for payments and refunds | 5 | ✅ |
| E5: Administration | Admin functions for platform oversight | 6 | ✅ |

## User Stories

### Epic 1: User Management

Authentication, authorization, and user profile management for the platform.

| ID | User Story | Acceptance Criteria | Priority | Status |
|----|------------|---------------------|----------|--------|
| US-001 | As a visitor, I want to register an account, so that I can access the booking platform | - Email must be unique<br>- Password minimum 8 characters<br>- Registered as User role by default | Must | ✅ |
| US-002 | As a user, I want to login with my credentials, so that I can access my account | - Returns JWT access token (60 min)<br>- Returns refresh token (7 days)<br>- Invalid credentials return 401 | Must | ✅ |
| US-003 | As a user, I want to refresh my access token, so that I can stay logged in | - Refresh token rotates on use<br>- Old refresh token is invalidated<br>- Returns new access and refresh tokens | Must | ✅ |
| US-004 | As a user, I want to become a hotel owner, so that I can list my properties | - Endpoint: POST /api/auth/become-hotel-owner<br>- Role changes from User to HotelOwner<br>- Returns new tokens with updated role | Must | ✅ |
| US-005 | As a user, I want to change my password, so that I can maintain account security | - Requires current password verification<br>- New password minimum 8 characters | Should | ✅ |
| US-006 | As a user, I want to logout, so that my session is terminated securely | - Refresh token is revoked<br>- Access token remains valid until expiry | Should | ✅ |
| US-007 | As a user, I want to request my data export, so that I comply with GDPR | - Creates export request<br>- Generates downloadable file<br>- File expires after period | Could | ✅ |
| US-008 | As an admin, I want to view all users, so that I can manage the platform | - Returns paginated user list<br>- Shows role and status<br>- Admin-only access | Must | ✅ |

### Epic 2: Hotel Management

Hotel and room CRUD operations with admin approval workflow.

| ID | User Story | Acceptance Criteria | Priority | Status |
|----|------------|---------------------|----------|--------|
| US-009 | As a hotel owner, I want to submit my hotel, so that it can be listed on the platform | - Hotel created with Pending status<br>- Includes location, description, cancellation policy<br>- Only HotelOwner role can submit | Must | ✅ |
| US-010 | As a hotel owner, I want to update my hotel details, so that information stays accurate | - Can only update own hotels<br>- Cannot update approval status<br>- Changes reflected immediately | Should | ✅ |
| US-011 | As a hotel owner, I want to add rooms to my hotel, so that guests can book them | - Room linked to approved hotel<br>- Includes capacity, price, accommodation type<br>- Room visibility can be toggled | Must | ✅ |
| US-012 | As a hotel owner, I want to view my hotels, so that I can manage my properties | - Returns only own hotels<br>- Shows approval status<br>- Includes room count | Must | ✅ |
| US-037 | As a hotel owner, I want to view reservations on my hotel, so that I can track bookings | - Returns all reservations for hotel's rooms<br>- Shows guest count, dates, status<br>- Only hotel owner or admin can access | Must | ✅ |
| US-013 | As a user, I want to search for hotels, so that I can find suitable accommodation | - Filter by country, city, dates<br>- Filter by pets allowed, price range<br>- Only returns approved hotels with available rooms | Must | ✅ |
| US-014 | As a user, I want to view hotel details, so that I can make booking decisions | - Shows full hotel information<br>- Includes cancellation policy<br>- Only visible for approved hotels | Must | ✅ |
| US-015 | As a user, I want to view room details, so that I can choose appropriate accommodation | - Shows capacity, price, amenities<br>- Shows accommodation type<br>- Only visible rooms returned | Must | ✅ |
| US-016 | As an admin, I want to view pending hotels, so that I can review submissions | - Returns hotels with Pending status<br>- Admin-only access | Must | ✅ |
| US-017 | As an admin, I want to approve hotels, so that they become visible to users | - Status changes to Approved<br>- ReviewedAt timestamp set<br>- Hotel becomes searchable | Must | ✅ |
| US-018 | As an admin, I want to reject hotels, so that unsuitable listings are excluded | - Status changes to Rejected<br>- ReviewedAt timestamp set<br>- Hotel not visible in search | Must | ✅ |

### Epic 3: Reservation Management

Booking lifecycle management with smart cancellation policies.

| ID | User Story | Acceptance Criteria | Priority | Status |
|----|------------|---------------------|----------|--------|
| US-019 | As a user, I want to create a reservation, so that I can book accommodation | - Reservation created with Pending status<br>- Room availability verified<br>- No overlapping dates allowed | Must | ✅ |
| US-020 | As a user, I want to view my reservations, so that I can track my bookings | - Returns only own reservations<br>- Shows status and dates<br>- Includes cancellation status | Must | ✅ |
| US-021 | As a user, I want to cancel my reservation, so that I can change my plans | - If within free period: auto-canceled with refund<br>- If outside free period: requires admin approval<br>- Cancellation reason required | Must | ✅ |
| US-022 | As an admin, I want to view all reservations, so that I can monitor bookings | - Returns paginated list<br>- Can filter by status<br>- Admin-only access | Should | ✅ |
| US-023 | As an admin, I want to view cancellation requests, so that I can process them | - Returns reservations with Requested status<br>- Shows cancellation reason<br>- Admin-only access | Must | ✅ |
| US-024 | As an admin, I want to approve cancellation requests, so that users get refunds | - Status changes to AdminApproved<br>- Triggers refund in Payments Service<br>- Reservation marked Canceled | Must | ✅ |
| US-025 | As an admin, I want to reject cancellation requests, so that policies are enforced | - Status changes to AdminRejected<br>- No refund processed<br>- Reservation remains Confirmed | Must | ✅ |

### Epic 4: Payment Processing

Stripe integration for secure payment processing.

| ID | User Story | Acceptance Criteria | Priority | Status |
|----|------------|---------------------|----------|--------|
| US-026 | As a user, I want to create a payment for my reservation, so that I can confirm booking | - Creates Stripe PaymentIntent<br>- Returns clientSecret for frontend<br>- Amount must match reservation | Must | ✅ |
| US-027 | As a user, I want to confirm my payment, so that my reservation is finalized | - Payment status updated via webhook<br>- Reservation status changes to Confirmed<br>- Payment details recorded | Must | ✅ |
| US-028 | As a user, I want to view my payment status, so that I can track transactions | - Shows payment status and amount<br>- Shows refund amount if applicable<br>- Only own payments visible | Should | ✅ |
| US-029 | As an admin, I want payments refunded automatically on cancellation approval, so that users receive their money | - Full refund processed via Stripe<br>- Payment status updated to Refunded<br>- RefundedAt timestamp recorded | Must | ✅ |
| US-030 | As a system, I want to receive Stripe webhooks, so that payment status is synchronized | - Webhook signature validated<br>- Payment status updated based on event<br>- Reservation service notified | Must | ✅ |

### Epic 5: Administration

Platform administration and oversight functions.

| ID | User Story | Acceptance Criteria | Priority | Status |
|----|------------|---------------------|----------|--------|
| US-031 | As an admin, I want to seed test data, so that I can demonstrate the platform | - Creates sample users, hotels, rooms<br>- Only runs if database is empty<br>- Uses configurable default password | Should | ✅ |
| US-032 | As a service, I want to authenticate internal API calls, so that services communicate securely | - API key required in X-API-Key header<br>- Each service has unique key<br>- Keys stored in Key Vault | Must | ✅ |
| US-033 | As an admin, I want to manually update reservation status, so that I can handle edge cases | - Can set status to any valid value<br>- Audit trail maintained<br>- Admin-only access | Could | ✅ |
| US-034 | As a service, I want health check endpoints, so that monitoring can verify service status | - Returns 200 OK when healthy<br>- Available without authentication | Should | ✅ |
| US-035 | As a developer, I want API documentation, so that I can understand endpoints | - Swagger UI available<br>- All endpoints documented<br>- Request/response examples included | Must | ✅ |
| US-036 | As an admin, I want to soft delete users, so that GDPR deletion is supported | - User marked as deleted<br>- Data anonymized or removed<br>- Cannot login after deletion | Should | ✅ |

## Use Case Diagram

```
                    ┌─────────────────────────────────────────────────────────┐
                    │              Hotel Booking Platform                      │
                    │                                                          │
    ┌───────┐       │  ┌─────────────────────────────────────────────────┐    │
    │       │       │  │  Register / Login / Manage Profile              │    │
    │ User  │───────┼──│  Search Hotels / View Hotel Details             │    │
    │       │       │  │  Create Reservation / Cancel Reservation        │    │
    └───────┘       │  │  Create Payment / View Payment Status           │    │
        │           │  └─────────────────────────────────────────────────┘    │
        │           │                                                          │
    ┌───────┐       │  ┌─────────────────────────────────────────────────┐    │
    │ Hotel │       │  │  Submit Hotel / Update Hotel                    │    │
    │ Owner │───────┼──│  Add Rooms / Update Rooms / Delete Rooms        │    │
    │       │       │  │  View Own Hotels / View Hotel Reservations      │    │
    └───────┘       │  └─────────────────────────────────────────────────┘    │
        │           │                                                          │
    ┌───────┐       │  ┌─────────────────────────────────────────────────┐    │
    │       │       │  │  Approve/Reject Hotels                          │    │
    │ Admin │───────┼──│  Approve/Reject Cancellations                   │    │
    │       │       │  │  View All Users / View All Reservations         │    │
    └───────┘       │  │  Manage Platform                                │    │
                    │  └─────────────────────────────────────────────────┘    │
                    │                                                          │
                    └─────────────────────────────────────────────────────────┘
```

## Non-Functional Requirements

### Performance

| Requirement | Target | Measurement Method |
|-------------|--------|-------------------|
| API response time | < 500ms for standard operations | Manual testing / Swagger |
| Concurrent users | Supports multiple concurrent API requests | Azure App Service scaling |
| Database queries | Optimized with proper indexing | EF Core query analysis |

### Security

- **Authentication**: JWT tokens with 60-minute expiry, refresh token rotation
- **Authorization**: Role-based access control (User, HotelOwner, Admin)
- **Data Protection**: PBKDF2 password hashing, HTTPS enforcement, Azure Key Vault for secrets
- **Input Validation**: DataAnnotations on all DTOs, EF Core parameterized queries
- **API Security**: Service-to-service API key authentication for internal endpoints

### Reliability

| Metric | Target |
|--------|--------|
| Service availability | 99%+ (Azure App Service SLA) |
| Data consistency | Eventual consistency between services |
| Error handling | Graceful degradation with proper error responses |

### Compatibility

| Platform/Tool | Minimum Version |
|---------------|-----------------|
| .NET Runtime | 9.0 |
| SQL Server | 2019+ / Azure SQL |
| Docker | 20.10+ |
| Swagger UI | Modern browsers (Chrome, Firefox, Safari, Edge) |
