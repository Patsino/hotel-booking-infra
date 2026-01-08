# Problem Statement & Goals

## Context

Modern hotel booking platforms have become increasingly complex, requiring users to navigate through multiple steps, pop-ups, and forms before completing a simple reservation. This friction in the booking process leads to cart abandonment and customer frustration. Additionally, pet owners face limited options when traveling, as most platforms don't adequately support pet-friendly accommodations or dedicated pet hotels.

## Problem Statement

**Who:** Travelers seeking quick hotel bookings without complex multi-step processes, pet owners needing accommodation for their pets, hotel owners wanting to list properties (including pet hotels), and platform administrators.

**What:** Modern booking platforms make the reservation process overly complex and multi-step, discouraging potential customers before they even reach payment. Users are forced through endless forms, account verifications, and upsell screens. Pet owners additionally struggle to find reliable pet-friendly hotels or dedicated pet care facilities when traveling.

**Why:** Complex booking flows lead to high abandonment rates. Users want to search, select, and pay—nothing more. The pet accommodation market remains underserved by mainstream platforms, leaving pet owners with limited visibility into available options.

### Pain Points

| # | Pain Point | Severity | Current Workaround |
|---|------------|----------|-------------------|
| 1 | **Overly complex booking flows** - Too many steps between finding a room and completing payment | High |Users abandon bookings or switch to simpler platforms |
| 2 | **Limited pet accommodation options** - Mainstream platforms don't highlight pet-friendly hotels or dedicated pet hotels | Medium | Pet owners call hotels directly or use fragmented pet-specific services |
| 3 | **Unclear cancellation policies** - Users don't know refund terms until deep in the booking process | Medium | Manual research or calling hotels directly |
| 4 | **Slow, cluttered interfaces** - Heavy platforms with ads, pop-ups, and unnecessary features | High | Desktop-only usage or frustration |


## Business Goals

| Goal | Description | Success Indicator |
|------|-------------|-------------------|
| Fast Booking Experience | Minimal steps from search to confirmed reservation | User can complete booking in under 2 minutes |
| Pet-Friendly Focus | Support for pet hotels and pet-friendly accommodations with clear filtering | Pet filter available, dedicated pet hotel category |
| Demonstrate Microservices Architecture | Build a fully functional system using 4 independent microservices | Four services deployed and communicating successfully |
| Implement Secure Authentication | Provide JWT-based authentication with role-based access control | Users can register, login, and access appropriate resources based on roles |
| Integrate Payment Processing | Implement Stripe payment integration with webhook handling | Payments can be created, confirmed, and refunded through Stripe |
| Cloud Deployment | Deploy working system to Azure cloud | All services accessible via public URLs |
| Docker for Local Development | Provide Docker setup for local development | docker-compose up starts full system |

## Objectives & Metrics

| Objective | Metric | Current Value | Target Value | Timeline |
|-----------|--------|---------------|--------------|----------|
| API Coverage | Automated test coverage | ≥70% | ≥70% | 2025-12-12 |
| Cloud Deployment | Services deployed to Azure | 4 | 4 | 2025-12-11 |
| Documentation | API documentation completeness | Swagger for all endpoints | Swagger for all endpoints | 2025-12-12 |

## Success Criteria

### Must Have

- [x] Four microservices (Users, Hotels, Reservations, Payments) deployed and functional
- [x] JWT-based authentication with refresh token rotation
- [x] Role-based access control (User, HotelOwner, Admin)
- [x] Stripe payment integration
- [x] ≥70% automated test coverage for all services
- [x] Azure cloud deployment (App Service + Key Vault)
- [x] Swagger API documentation
- [x] Docker Compose for local development
- [x] Database migrations via Entity Framework Core

### Nice to Have

- [x] React frontend application for user-friendly booking interface
- [ ] CI/CD pipeline (not implemented - manual deployment via Visual Studio)
- [ ] Message queue for async communication (using synchronous HTTP)

## Non-Goals

What this project explicitly does NOT aim to achieve:

- Real-time messaging or notifications (email, SMS, push notifications)
- Mobile application development
- Multi-language/localization support
- Review and rating system for hotels
- Image upload and storage functionality
- Full production-grade monitoring and alerting infrastructure
