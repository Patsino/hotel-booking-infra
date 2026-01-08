# 2. Technical Implementation

This section covers the technical architecture, design decisions, and implementation details.

## Contents

- [Tech Stack](tech-stack.md)
- [Criteria Documentation](criteria/) - ADR for each evaluation criterion
- [Deployment](deployment.md)


## Solution Architecture

### High-Level Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                     React Frontend                                  │
│                  http://localhost:5173                              │
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
     │    │    Service-to-Service Communication
     │    │    (REST + API Keys)
     ▼    ▼
  ┌───────────────────────────────────┐    ┌──────────────┐
  │   SQL Server: HotelBooking DB     │    │  Stripe API  │
  │  ┌──────┬────────┬────────┬──────┐│    │  (Payments)  │
  │  │users │hotels  │reserv. │paym  ││    └──────────────┘
  │  │schema│schema  │schema  │schema││
  │  └──────┴────────┴────────┴──────┘│
  └───────────────────────────────────┘
```

### System Components

| Component | Description | Technology |
|-----------|-------------|------------|
| **Frontend** | React SPA for user-friendly booking interface | React, TypeScript, Vite |
| **Users Service** | Authentication, user management, GDPR compliance | ASP.NET Core, JWT, PBKDF2 |
| **Hotels Service** | Hotel/room management, search, approval workflow | ASP.NET Core, EF Core |
| **Reservations Service** | Booking lifecycle, cancellation policies | ASP.NET Core, EF Core |
| **Payments Service** | Stripe integration, payment processing, webhooks | ASP.NET Core, Stripe.NET |
| **Database** | Shared SQL Server with schema separation | SQL Server / Azure SQL |
| **Secret Management** | Centralized secrets and configuration | Azure Key Vault |

### Data Flow

```
[User Action] → [API Request] → [Service Controller]
                                        │
                                        ▼
                                 [Command/Query Handler]
                                        │
                                        ▼
                          [Domain Logic / Business Rules]
                                        │
                          ┌─────────────┴─────────────┐
                          ▼                           ▼
                   [Repository]              [External Service Call]
                          │                    (Other Microservice
                          ▼                     or Stripe API)
                   [EF Core DbContext]
                          │
                          ▼
                   [SQL Server]
                          │
                          ▼
                   [API Response]
                          │
[UI Update] ←─────────────┘
```

## Key Technical Decisions

| Decision | Rationale | Alternatives Considered |
|----------|-----------|------------------------|
| Microservices Architecture | Independent deployment, scaling, and development of each bounded context | Monolithic (rejected: harder to scale and maintain) |
| Shared Database with Schemas | Azure free tier limitation allows only one database; schemas provide logical separation | Separate databases (rejected: not feasible in free tier) |
| JWT Authentication | Stateless, scalable authentication suitable for microservices | Session-based (rejected: requires shared state) |
| API Key for Service-to-Service | Simple, secure internal authentication | OAuth2 (rejected: added complexity for internal calls) |
| Stripe for Payments | Industry standard, comprehensive API, test mode support | Manual payment handling (rejected: security concerns) |

## Security Overview

| Aspect | Implementation |
|--------|----------------|
| **Authentication** | JWT tokens (60 min expiry) with refresh token rotation (7 days) |
| **Authorization** | Role-based (User, HotelOwner, Admin) with policy-based checks |
| **Data Protection** | PBKDF2 password hashing, TLS/HTTPS enforcement, Azure Key Vault for secrets |
| **Secrets Management** | Azure Key Vault with Managed Identity access, environment variables in Docker |
| **API Security** | Service-to-service API keys, webhook signature validation (Stripe) |
