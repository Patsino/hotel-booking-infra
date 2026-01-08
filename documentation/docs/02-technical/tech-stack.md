# Technology Stack

## Stack Overview

| Layer | Technology | Version | Justification |
|-------|------------|---------|---------------|
| **Frontend** | React | 18.x | Component-based UI with fast development cycle |
| **Frontend Build** | Vite | 5.x | Fast build tool with HMR for React |
| **Frontend Language** | TypeScript | 5.x | Type safety for frontend development |
| **Runtime** | .NET | 9.0 | Latest LTS with performance improvements, native AOT support |
| **Framework** | ASP.NET Core | 9.0 | Industry-standard for building RESTful APIs in .NET |
| **ORM** | Entity Framework Core | 9.0 | First-class .NET integration, migrations, LINQ support |
| **Database** | SQL Server / Azure SQL | 2019+ | Robust relational database with Azure integration |
| **Authentication** | JWT Bearer Tokens | - | Stateless authentication suitable for microservices |
| **Payment Processing** | Stripe | - | Industry-standard payment processor with comprehensive API |
| **Containerization** | Docker | 20.10+ | Consistent development and deployment environments |
| **Cloud Platform** | Azure | - | App Service, SQL Database, Key Vault integration |
| **API Documentation** | Swagger/OpenAPI | 3.0 | Interactive API documentation and testing |

## Frontend

**Repository:** https://github.com/Patsino/hotel-booking-frontend

The React frontend provides a lightweight, user-friendly booking interface designed for minimal-effort reservations.

**Key Libraries:**
- React 18 with functional components and hooks
- TypeScript for type safety
- React Router for navigation
- Axios for API communication
- Stripe Elements for payment UI

## Key Technology Decisions

### Decision 1: .NET 9 with ASP.NET Core

**Context:** Need a robust, performant backend framework for building RESTful microservices.

**Decision:** Use .NET 9 with ASP.NET Core for all four microservices.

**Rationale:**
- Latest .NET version with performance improvements
- Strong typing and compile-time safety
- Excellent tooling support (Visual Studio, VS Code)
- Built-in dependency injection
- Rich ecosystem for authentication, ORM, and testing

**Trade-offs:**
- Pros: Performance, type safety, comprehensive framework features, strong Azure integration
- Cons: Larger runtime footprint compared to minimal frameworks

### Decision 2: Entity Framework Core with Code-First Migrations

**Context:** Need an ORM for database operations with support for migrations and multiple database providers.

**Decision:** Use Entity Framework Core with code-first approach.

**Rationale:**
- Seamless integration with ASP.NET Core
- Type-safe queries with LINQ
- Automatic migration generation
- Support for both SQL Server and In-Memory provider for testing

**Trade-offs:**
- Pros: Developer productivity, type safety, migration support, testability
- Cons: Performance overhead compared to raw SQL for complex queries

### Decision 3: JWT with Refresh Token Rotation

**Context:** Need stateless authentication that works across multiple microservices.

**Decision:** Implement JWT access tokens (60 min) with refresh token rotation (7 days).

**Rationale:**
- Stateless authentication eliminates shared session state
- Short-lived access tokens limit exposure if compromised
- Refresh token rotation provides security with convenience
- Standard claims (sub, role, email) for authorization decisions

**Trade-offs:**
- Pros: Scalability, no shared state, industry standard
- Cons: Token revocation requires additional infrastructure (refresh token blacklist)

### Decision 4: Shared Database with Schema Separation

**Context:** Azure free tier allows only one SQL database, but microservices need logical data separation.

**Decision:** Use a single SQL Server database with separate schemas per service.

**Rationale:**
- Complies with Azure free tier limitations
- Logical separation via schemas (users, hotels, reservations, payments)
- Each service owns its schema exclusively
- No cross-schema foreign keys (maintained in application layer)

**Trade-offs:**
- Pros: Cost-effective, still maintains logical boundaries
- Cons: Not true database-per-service isolation

## Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| **IDE** | Visual Studio 2022 / VS Code | C# Dev Kit extension for VS Code |
| **Version Control** | Git | GitHub repositories per microservice |
| **Package Manager** | NuGet | Standard .NET package management |
| **Testing** | xUnit, FluentAssertions, Moq | >70% coverage target |
| **API Testing** | Swagger UI, .http files | Built-in request testing |
| **Coverage** | Coverlet + ReportGenerator | HTML coverage reports |
| **Containerization** | Docker, Docker Compose | Local development orchestration |

## External Services & APIs

| Service | Purpose | Pricing Model |
|---------|---------|---------------|
| **Stripe** | Payment processing (intents, confirmations, refunds, webhooks) | Pay per transaction (using test mode) |
| **Azure App Service** | Web application hosting | Free F1 tier (student subscription) |
| **Azure SQL Database** | Managed SQL Server database | Free tier - General Purpose Serverless |
| **Azure Key Vault** | Secrets and configuration management | Standard tier |

## NuGet Packages

### Core Packages (All Services)

| Package | Purpose |
|---------|---------|
| `Microsoft.AspNetCore.Authentication.JwtBearer` | JWT authentication |
| `Microsoft.EntityFrameworkCore.SqlServer` | SQL Server database provider |
| `Swashbuckle.AspNetCore` | Swagger/OpenAPI documentation |
| `Azure.Identity` | Azure Key Vault authentication |
| `Azure.Extensions.AspNetCore.Configuration.Secrets` | Key Vault configuration provider |

### Testing Packages

| Package | Purpose |
|---------|---------|
| `xunit` | Testing framework |
| `FluentAssertions` | Expressive assertions |
| `Moq` | Mocking framework |
| `Microsoft.AspNetCore.Mvc.Testing` | Integration testing |
| `Microsoft.EntityFrameworkCore.InMemory` | In-memory database for tests |
| `coverlet.collector` | Code coverage collection |

### Service-Specific Packages

| Service | Package | Purpose |
|---------|---------|---------|
| Payments | `Stripe.net` | Stripe API client |
| Users | `Microsoft.AspNetCore.Cryptography.KeyDerivation` | PBKDF2 password hashing |
