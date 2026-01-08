# Criterion 5: Microservices Architecture

## Architecture Decision Record

### Status

**Status:** Accepted

**Date:** 2025-12-12

### Context

The project demonstrates microservices architecture patterns for a complex domain (hotel booking) that naturally decomposes into distinct bounded contexts. The architecture must enable:
- Independent development and deployment of services
- Clear domain boundaries
- Secure inter-service communication
- Fault isolation

### Decision

Implement four microservices following Domain-Driven Design bounded contexts:
1. **Users Service** - Identity & Access Management
2. **Hotels Service** - Hotel & Room Inventory
3. **Reservations Service** - Booking Management
4. **Payments Service** - Payment Processing

Each service:
- Owns its data (separate database schema)
- Exposes REST APIs for external clients
- Exposes internal APIs for service-to-service communication
- Uses API key authentication for internal calls

### Alternatives Considered

| Alternative | Pros | Cons | Why Not Chosen |
|-------------|------|------|----------------|
| Monolithic | Simpler, easier debugging | Hard to scale, tight coupling | Does not meet criteria |


### Consequences

**Positive:**
- Clear separation of business domains
- Independent scaling potential
- Technology flexibility per service (all .NET, but could differ)
- Fault isolation between services
- Demonstrates real-world architecture patterns

**Negative:**
- Distributed system complexity
- Network latency between services
- Data consistency challenges
- More deployment targets to manage

**Neutral:**
- Each service needs its own testing and deployment pipeline

## Implementation Details

### Bounded Contexts

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Hotel Booking Platform                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────┐    ┌──────────────────────┐                       │
│  │  Identity & Access   │    │   Hotel Management   │                       │
│  │    (Users Service)   │    │   (Hotels Service)   │                       │
│  │                      │    │                      │                       │
│  │  - User              │    │  - Hotel             │                       │
│  │  - RefreshToken      │    │  - Room              │                       │
│  │  - Role              │    │  - ApprovalStatus    │                       │
│  │  - DataExportRequest │    │  - AccommodationType │                       │
│  └──────────────────────┘    └──────────────────────┘                       │
│                                                                              │
│  ┌──────────────────────┐    ┌──────────────────────┐                       │
│  │ Booking Management   │    │  Payment Processing  │                       │
│  │(Reservations Service)│    │  (Payments Service)  │                       │
│  │                      │    │                      │                       │
│  │  - Reservation       │    │  - Payment           │                       │
│  │  - ReservationStatus │    │  - PaymentStatus     │                       │
│  │  - CancellationStatus│    │  - PaymentProvider   │                       │
│  └──────────────────────┘    └──────────────────────┘                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Service Communication

```
┌──────────────────┐                    ┌──────────────────┐
│   Client/User    │                    │   Stripe API     │
└────────┬─────────┘                    └────────┬─────────┘
         │ JWT Token                             │ Webhooks
         ▼                                       ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                              API Gateway (Logical)                            │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐ │
│  │   Users     │     │   Hotels    │     │ Reservations│     │  Payments   │ │
│  │   :8081     │     │   :8082     │     │   :8083     │     │   :8084     │ │
│  └──────┬──────┘     └──────┬──────┘     └──────┬──────┘     └──────┬──────┘ │
│         │                   │                   │                   │        │
│         └───────────────────┴───────────────────┴───────────────────┘        │
│                           │ Internal API Calls (REST + API Key)              │
│                           ▼                                                   │
│              ┌─────────────────────────────┐                                 │
│              │    SQL Server Database      │                                 │
│              │  (Schema per service)       │                                 │
│              └─────────────────────────────┘                                 │
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
```

### Internal API Endpoints

Each service exposes `/internal/*` endpoints for service-to-service communication:

**Users Service:**
```
GET  /api/internal-users/{id}      - Get user details
POST /api/internal-users/validate  - Validate user exists
```

**Hotels Service:**
```
GET  /api/internal/hotels/{id}              - Get hotel details
GET  /api/internal/rooms/{id}               - Get room details
GET  /api/internal/rooms/{id}/availability  - Check availability
POST /api/internal/rooms/validate           - Validate room bookable
```

**Reservations Service:**
```
PATCH /api/internal/reservations/{id}/confirm  - Confirm reservation
GET   /api/internal/reservations/{id}          - Get reservation
POST  /api/internal/reservations/validate      - Validate reservation
```

**Payments Service:**
```
POST /api/internal/payments/refund  - Process refund
GET  /api/internal/payments/{id}    - Get payment details
```

### API Key Authentication

Internal endpoints use API key authentication:

```csharp
public class ApiKeyAuthenticationHandler : AuthenticationHandler<ApiKeyAuthenticationOptions>
{
    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        if (!Request.Headers.TryGetValue("X-API-Key", out var apiKey))
            return Task.FromResult(AuthenticateResult.NoResult());

        var validApiKey = _configuration[$"ApiKeys:Services:{_serviceName}"];
        
        if (apiKey != validApiKey)
            return Task.FromResult(AuthenticateResult.Fail("Invalid API key"));

        var claims = new[] { new Claim(ClaimTypes.Name, "InternalService") };
        var identity = new ClaimsIdentity(claims, Scheme.Name);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Scheme.Name);
        
        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

### Service Dependencies

| Service | Depends On | Communication |
|---------|------------|---------------|
| Users | None | - |
| Hotels | Users (validates owner) | HTTP + API Key |
| Reservations | Hotels (room availability) | HTTP + API Key |
| Payments | Reservations (confirm booking) | HTTP + API Key |

### Data Consistency

**Strategy: Eventual Consistency**

The system uses eventual consistency between services:

1. **Payment → Reservation Confirmation:**
   - User creates reservation (Pending)
   - User creates payment intent
   - Stripe processes payment
   - Webhook triggers reservation confirmation
   - Reservation becomes Confirmed

2. **Cancellation → Refund:**
   - User requests cancellation
   - Admin approves (if needed)
   - Reservations Service calls Payments Service
   - Stripe processes refund
   - Both records updated

**Compensating Transactions:**
- Failed payment: Reservation stays Pending, can retry
- Failed refund: Payment marked for manual review

## Requirements Checklist

### Minimum Requirements (Grade 5)

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Architecture diagram (services, protocols) | ✅ | Diagrams in this document and reference/ |
| 2 | Domain model with bounded contexts | ✅ | Users, Hotels, Reservations, Payments |
| 3 | API documentation (contracts) | ✅ | Swagger/OpenAPI for all services |
| 4 | Deployment diagram | ✅ | docs/02-technical/deployment.md |
| 5 | Decomposition justification | ✅ | Natural domain boundaries |
| 6 | Minimum 3 business services | ✅ | Four services (not counting infra) |
| 7 | Loose coupling | ✅ | Each service independent |
| 8 | No shared database (own DB/schema) | ✅ | Separate schemas per service |
| 9 | Synchronous communication (REST/gRPC) | ✅ | REST with API keys |
| 10 | Timeouts and retry policies | ✅ | Polly retry (3 attempts, exponential backoff) + 5s timeout policy per request |
| 11 | Health/ready endpoints | ✅ | /health endpoint on each service |
| 12 | Correlation ID logging | ✅ | CorrelationIdMiddleware generates/propagates X-Correlation-ID header |
| 13 | API Gateway or entry point | ✅ | JWT auth acts as logical gateway - single auth mechanism, services validate tokens |
| 14 | Unit tests per service | ✅ | Tests project for each service |
| 15 | Integration tests | ✅ | Tests.Integration projects |

### Maximum Requirements (Grade 10) - Optional Directions

| Direction | Status | Notes |
|-----------|--------|-------|
| Async communication (message broker) | ❌ | HTTP only - not required for minimum |
| Resilience patterns (circuit breaker) | ❌ | No Polly or resilience patterns |
| Observability (distributed tracing) | ❌ | No Jaeger/Zipkin |
| Advanced patterns (Saga, CQRS) | ❌ | Simple handler pattern |
| API Management (Kong, rate limiting) | ❌ | Direct service access |

**Note:** Maximum requirements are optional enhancements. Minimum requirements fully satisfied.

## Known Limitations

| Limitation | Impact | Potential Solution |
|------------|--------|-------------------|
| No message queue | Synchronous coupling | Add RabbitMQ/Azure Service Bus |
| No circuit breaker | Cascade failures possible | Implement Polly policies |
| No service mesh | Limited observability | Add Istio or similar |
| Configuration-based discovery | Manual URL updates | Implement service registry |

## References

- [Microservices by Martin Fowler](https://martinfowler.com/articles/microservices.html)
- [Domain-Driven Design](https://docs.microsoft.com/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/)
- Internal controllers: `Api/Controllers/Internal*.cs` in each service
- HTTP clients: `Infrastructure/Http/` in each service
