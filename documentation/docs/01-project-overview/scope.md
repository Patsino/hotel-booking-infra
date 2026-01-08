# Project Scope

## In Scope ✅

| Feature | Description | Priority |
|---------|-------------|----------|
| User Authentication | Registration, login, JWT tokens, refresh token rotation | Must |
| Role-Based Access Control | User, HotelOwner, Admin roles with appropriate permissions | Must |
| Hotel Management | Create, update, search hotels with approval workflow | Must |
| Room Management | Create, update, delete rooms with pricing and availability | Must |
| Reservation Management | Create reservations, view bookings, cancellation workflow | Must |
| Payment Processing | Stripe integration for payment intents, confirmation, refunds | Must |
| Admin Functions | Hotel approval, cancellation request handling, user management | Must |
| API Documentation | Swagger/OpenAPI documentation for all endpoints | Must |
| Automated Testing | Unit and integration tests with ≥70% coverage | Must |
| Cloud Deployment | Azure App Service deployment with Key Vault secrets | Must |
| Docker Support | Docker Compose for local development environment | Must |
| GDPR Compliance | Data export requests, user data deletion | Must |
| Service-to-Service Auth | API key authentication for internal endpoints | Must |

## Out of Scope ❌

| Feature | Reason | When Possible |
|---------|--------|---------------|
| Mobile Application | Focus on backend microservices architecture | Future phase |
| Review/Rating System | Not essential for core booking flow demonstration | Future phase |
| Email/SMS Notifications | Requires additional infrastructure (SendGrid, Twilio) | Future phase |
| Image Upload/Storage | Would require Azure Blob Storage integration | Future phase |
| Multi-Language Support | Localization adds complexity without demonstrating core concepts | Future phase |
| Loyalty/Rewards Program | Business feature beyond core booking functionality | Future phase |
| Advanced Analytics | Would require additional data warehouse infrastructure | Future phase |

## Assumptions

| # | Assumption | Impact if Wrong | Probability |
|---|------------|-----------------|-------------|
| 1 | Azure free tier resources sufficient for demonstration | May need to upgrade or limit testing | Low |
| 2 | Stripe test mode adequate for payment flow demonstration | Would need production keys and compliance | Low |
| 3 | Single SQL Server database with schema separation acceptable | Would need separate database instances | Medium |
| 4 | Users have modern web browsers supporting Swagger UI | May need alternative documentation format | Low |
| 5 | JWT tokens sufficient for stateless authentication | May need to implement additional session management | Low |

## Constraints

Limitations that affect the project:

| Constraint Type | Description | Mitigation |
|-----------------|-------------|------------|
| **Time** | Diploma project deadline 2026-01-07 | Prioritize must-have features, defer nice-to-haves |
| **Budget** | Azure student subscription with free tier limits | Use free tier resources, optimize for cost |
| **Technology** | Required to use .NET for backend services | Leverage .NET 9 latest features and best practices |
| **Resources** | Single developer working on entire system | Focus on clean architecture for maintainability |
| **External** | Stripe payment processing dependency | Use test mode, handle webhook failures gracefully |
| **Infrastructure** | Azure student subscription allows only one free SQL database | Split single database into schemas per microservice |

## Dependencies

| Dependency | Type | Owner | Status |
|------------|------|-------|--------|
| Stripe API | External | Stripe | ✅ Active |
| Azure App Service | External | Microsoft | ✅ Active |
| Azure SQL Database | External | Microsoft | ✅ Active |
| Azure Key Vault | External | Microsoft | ✅ Active |
| .NET 9 SDK | Technical | Microsoft | ✅ Active |
| Entity Framework Core | Technical | Microsoft | ✅ Active |
| SQL Server (Docker) | Technical | Microsoft | ✅ Active |
