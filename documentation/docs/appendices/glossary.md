# Glossary

## General Terms

| Term | Definition |
|------|------------|
| Access Token | Short-lived JWT token (60 minutes) used to authenticate API requests |
| Refresh Token | Long-lived token (7 days) used to obtain new access tokens without re-login |
| Microservice | An independently deployable service responsible for a specific business capability |
| Clean Architecture | Software design pattern separating code into layers (Domain, Application, Infrastructure, API) |
| Code-First | EF Core approach where database schema is generated from C# entity classes |
| Webhook | HTTP callback triggered by external service (Stripe) to notify of events |
| Payment Intent | Stripe object representing a payment lifecycle from creation to completion |

## Acronyms

| Acronym | Full Form | Description |
|---------|-----------|-------------|
| API | Application Programming Interface | Contract defining how services communicate |
| JWT | JSON Web Token | Compact, URL-safe token for secure claims transmission |
| EF | Entity Framework | Microsoft's ORM for .NET applications |
| ORM | Object-Relational Mapping | Technique mapping objects to database tables |
| GUID | Globally Unique Identifier | 128-bit identifier used as primary keys |
| CORS | Cross-Origin Resource Sharing | Security feature controlling cross-domain requests |
| CI/CD | Continuous Integration/Continuous Deployment | Automated build and deployment pipeline |
| SPA | Single Page Application | Web app loading a single HTML page dynamically |
| REST | Representational State Transfer | Architectural style for web services |
| DTO | Data Transfer Object | Object carrying data between processes |
| DI | Dependency Injection | Design pattern for providing dependencies |

## Domain-Specific Terms

### Hotel & Booking Domain

| Term | Definition |
|------|------------|
| Hotel | A property offering rooms for accommodation |
| Room | An individual bookable unit within a hotel |
| Room Type | Category of room (Standard, Deluxe, Suite, etc.) |
| Accommodation Type | Type of accommodation: HotelRoom, Apartment, House, Cabin, or Capsule |
| Reservation | A booking request for a room during specific dates |
| Check-in Date | The date a guest arrives at the hotel |
| Check-out Date | The date a guest departs from the hotel |
| Guest Count | Number of people staying in the room |
| Hotel Owner | User role that can create and manage hotels |
| Free Cancellation Period | Days before check-in when cancellation incurs no charge |

### Reservation Status

| Status | Definition |
|--------|------------|
| Pending | Reservation created, awaiting payment |
| Confirmed | Payment received, booking secured |
| CancellationPending | Cancellation requested, awaiting admin approval |
| Cancelled | Reservation successfully cancelled |
| Completed | Guest stay finished |
| NoShow | Guest did not arrive for reservation |

### Payment Terms

| Term | Definition |
|------|------------|
| Payment Intent | Stripe object tracking payment from creation to completion |
| Client Secret | Token used by frontend to complete payment securely |
| Refund | Return of payment amount to customer |
| Stripe | Third-party payment processing platform |
| Test Card | Stripe test card number (4242...) for sandbox testing |

### User Roles

| Role | Definition |
|------|------------|
| User | Standard customer who can search hotels and make reservations |
| HotelOwner | Business user who can list hotels and manage rooms |
| Admin | System administrator with full access to all operations |

### Architecture Terms

| Term | Definition |
|------|------------|
| Domain Layer | Core business logic and entities, no external dependencies |
| Application Layer | Use cases, commands, queries, and service interfaces |
| Infrastructure Layer | External concerns: database, HTTP clients, third-party services |
| API Layer | HTTP endpoints, controllers, and request/response handling |
| Handler | Class implementing a specific command or query operation |
| Command | Object representing an action that modifies state |
| Query | Object representing a request for data |

### Azure Services

| Term | Definition |
|------|------------|
| Azure App Service | PaaS for hosting web applications and APIs |
| Azure SQL | Managed SQL Server database service |
| Azure Key Vault | Secure storage for secrets, keys, and certificates |
| Free Tier (F1) | Azure's no-cost hosting plan with limited resources |
| Cold Start | Delay when app starts after period of inactivity on free tier |

### Testing Terms

| Term | Definition |
|------|------------|
| Unit Test | Test verifying individual component in isolation |
| Integration Test | Test verifying multiple components working together |
| Mock | Simulated object replacing real dependency in tests |
| Code Coverage | Percentage of code executed by tests |
| xUnit | .NET testing framework used in this project |
| FluentAssertions | Library for readable test assertions |
