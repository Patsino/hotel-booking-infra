# Criterion 1: Back-end

## Architecture Decision Record

### Status

**Status:** Accepted

**Date:** 2025-12-12

### Context

The project requires a robust back-end architecture capable of handling multiple business domains (users, hotels, reservations, payments) with proper separation of concerns, scalability, and maintainability. The back-end must support RESTful APIs, secure authentication, and integration with external services (Stripe).

### Decision

Implement a microservices architecture using .NET 9 with ASP.NET Core, following Clean Architecture principles within each service. Each microservice follows a layered structure:
- **Api Layer**: Controllers, middleware, filters
- **Application Layer**: Commands, queries, handlers, DTOs
- **Domain Layer**: Entities, value objects, domain logic
- **Infrastructure Layer**: Repositories, external services, database context

### Alternatives Considered

| Alternative | Pros | Cons | Why Not Chosen |
|-------------|------|------|----------------|
| Monolithic Architecture | Simpler deployment, easier debugging | Harder to scale, tight coupling, difficult to maintain | Does not demonstrate microservices patterns |

### Consequences

**Positive:**
- Clear separation of concerns between services
- Independent deployment and scaling of each service
- Type-safe code with compile-time error checking
- Rich .NET ecosystem for authentication, ORM, testing

**Negative:**
- Increased complexity in service communication
- Need for distributed tracing and logging
- More infrastructure to manage

**Neutral:**
- Learning curve for Clean Architecture patterns

## Implementation Details

### Project Structure

Each microservice follows this structure:

```
hotel-booking-{service}-service/
├── Api/
│   ├── Controllers/           # REST API endpoints
│   ├── Filters/               # Exception filters
│   ├── Middleware/            # Request pipeline middleware
│   └── Program.cs             # Application entry point
├── Application/
│   ├── Commands/              # Write operations
│   ├── Dtos/                  # Data transfer objects
│   ├── Handlers/              # Command/query handlers
│   └── Services/              # Application services
├── Domain/
│   ├── {Entity}/              # Domain entities
│   │   ├── {Entity}.cs        # Entity definition
│   │   └── I{Entity}Repository.cs  # Repository interface
│   └── Enums/                 # Domain enums
├── Infrastructure/
│   ├── Authentication/        # JWT/API key auth
│   ├── Authorization/         # Policy handlers
│   ├── Http/                  # HTTP clients
│   ├── Migrations/            # EF Core migrations
│   ├── Persistence/           # DbContext, repositories
│   └── DependencyInjection.cs # Service registration
└── Tests/
    └── Unit/Integration tests
```

### Key Implementation Decisions

| Decision | Rationale |
|----------|-----------|
| Clean Architecture | Separation of concerns, testability, dependency inversion |
| Repository pattern | Abstraction over data access via interfaces (IUsersRepository, etc.) |
| Handler pattern | Each use case implemented as separate handler class (RegisterUserHandler, etc.) |
| Dependency Injection | Built-in .NET DI container for loose coupling |
| Async database operations | All EF Core calls use async methods (ToListAsync, SaveChangesAsync) |

### Service Overview

| Service | Port | Responsibility |
|---------|------|----------------|
| Users | 8081 | Authentication, user management, GDPR |
| Hotels | 8082 | Hotel/room management, search, approval workflow |
| Reservations | 8083 | Booking lifecycle, cancellation policies |
| Payments | 8084 | Stripe integration, payment processing |

### Code Examples

**Controller Example (Users Service):**
```csharp
[ApiController]
[Route("api/auth")]
public sealed class AuthController : ControllerBase
{
    private readonly IRegisterUserHandler _registerHandler;
    private readonly ILoginHandler _loginHandler;

    [HttpPost("register")]
    [ProducesResponseType(StatusCodes.Status201Created)]
    public async Task<IActionResult> Register([FromBody] RegisterUserCommand command)
    {
        var result = await _registerHandler.HandleAsync(command);
        return CreatedAtAction(nameof(Register), new { id = result.UserId }, result);
    }
}
```

**Handler Example:**
```csharp
public class RegisterUserHandler : IRegisterUserHandler
{
    private readonly IUsersRepository _repository;
    private readonly IPasswordHasher _passwordHasher;

    public async Task<RegisterUserResult> HandleAsync(RegisterUserCommand command)
    {
        // Validate email uniqueness
        var existingUser = await _repository.GetByEmailAsync(command.Email);
        if (existingUser != null)
            throw new InvalidOperationException("Email already registered");

        // Create user with hashed password
        var user = User.Create(
            command.Email,
            _passwordHasher.HashPassword(command.Password),
            command.FirstName,
            command.LastName,
            command.Role);

        await _repository.AddAsync(user);
        return new RegisterUserResult(user.Id, user.Email, user.Role);
    }
}
```

## Requirements Checklist

### Minimum Requirements (Grade 5)

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Modern framework (ASP.NET Core, Spring, Django, etc.) | ✅ | .NET 9 with ASP.NET Core |
| 2 | Database for state management | ✅ | SQL Server via Entity Framework Core |
| 3 | ORM usage | ✅ | Entity Framework Core with code-first |
| 4 | Layered architecture | ✅ | Api/Application/Domain/Infrastructure layers |
| 5 | SOLID principles | ✅ | DI, single responsibility handlers, interface segregation |
| 6 | API documentation | ✅ | Swagger/OpenAPI with XML comments |
| 7 | Global error handling | ✅ | ExceptionHandlingMiddleware in each service |
| 8 | Logging | ✅ | ILogger with structured logging |
| 9 | Production deployment | ✅ | Azure App Service (North Europe) |
| 10 | Test coverage ≥70% | ✅ | 75-85% coverage per service |

### Maximum Requirements (Grade 10)

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 1 | CI/CD pipeline | ❌ | Not implemented - manual deployment via Visual Studio. CI/CD not selected as criterion |
| 2 | Microservices architecture | ✅ | Four independent services |

## Known Limitations

| Limitation | Impact | Potential Solution |
|------------|--------|-------------------|
| No message queue for async communication | Services rely on synchronous HTTP calls | Implement RabbitMQ or Azure Service Bus |
| No distributed tracing | Debugging cross-service issues harder | Add correlation ID logging (partially implemented) |
| No circuit breaker pattern | Service failures can cascade | Implement Polly retry/circuit breaker policies |

## References

- [ASP.NET Core Documentation](https://docs.microsoft.com/aspnet/core)
- [Clean Architecture by Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- Controllers: `Api/Controllers/` in each service
- Handlers: `Application/Handlers/` in each service
