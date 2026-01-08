# Criterion 2: API Documentation

## Architecture Decision Record

### Status

**Status:** Accepted

**Date:** 2025-12-12

### Context

The project requires comprehensive API documentation that allows developers and reviewers to understand, test, and integrate with the microservices. Documentation must be interactive, always up-to-date with the code, and provide clear examples of request/response formats.

### Decision

Implement Swagger/OpenAPI 3.0 documentation using Swashbuckle.AspNetCore, with:
- XML documentation comments on all controllers and methods
- Swagger annotations for operation metadata
- Example schemas for request/response bodies
- Authentication support in Swagger UI
- Separate Swagger instance per microservice

### Alternatives Considered

| Alternative | Pros | Cons | Why Not Chosen |
|-------------|------|------|----------------|
| Manual documentation (Markdown) | Full control over format | Quickly becomes outdated, no interactive testing | Not synchronized with code |
| Postman Collections | Good for testing, shareable | Separate from code, manual maintenance | Not integrated with deployment |

### Consequences

**Positive:**
- Auto-generated from code annotations - always up-to-date
- Interactive testing directly in browser
- Standard OpenAPI format exportable to other tools
- Clear endpoint documentation with examples

**Negative:**
- Requires maintaining XML comments and annotations
- Swagger UI adds to application size

**Neutral:**
- Developers must follow documentation conventions

## Implementation Details

### Swagger Configuration

Each service configures Swagger in `Program.cs`:

```csharp
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Hotel Booking - Users Service API",
        Version = "v1",
        Description = "Authentication and user management microservice"
    });

    // JWT Authentication in Swagger
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Type = SecuritySchemeType.Http,
        Scheme = "bearer",
        BearerFormat = "JWT",
        Description = "Enter JWT token"
    });

    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    options.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFile));
});
```

### Documentation Standards

**Controller Documentation:**
```csharp
/// <summary>
/// Create Stripe payment intent for a reservation
/// </summary>
/// <param name="command">Payment details including reservation ID, amount, and currency</param>
/// <returns>Payment intent with client secret for frontend confirmation</returns>
/// <remarks>
/// Creates a Stripe payment intent. Returns client secret for confirming payment on frontend.
/// 
/// Sample request:
/// 
///     POST /api/payments/create-intent
///     {
///        "reservationId": 150,
///        "amount": 178.00,
///        "currency": "EUR"
///     }
/// 
/// **Response includes:**
/// - **paymentIntentId**: Stripe payment intent ID
/// - **clientSecret**: Use with Stripe.js to confirm payment
/// - **paymentId**: Internal payment record ID
/// </remarks>
/// <response code="200">Payment intent created successfully</response>
/// <response code="400">Invalid reservation or amount</response>
/// <response code="401">User not authenticated</response>
/// <response code="403">User trying to pay for another user's reservation</response>
[HttpPost("create-intent")]
[SwaggerOperation(Summary = "Create payment intent", Description = "Create Stripe payment intent")]
[SwaggerResponse(200, "Payment intent created")]
[SwaggerResponse(400, "Invalid request")]
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async Task<IActionResult> CreatePaymentIntent([FromBody] CreatePaymentIntentCommand command)
```

### API Endpoints by Service

#### Users Service (Port 8081)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/register` | POST | Register new user |
| `/api/auth/login` | POST | Login and get tokens |
| `/api/auth/refresh` | POST | Refresh access token |
| `/api/auth/logout` | POST | Revoke refresh token |
| `/api/users` | GET | Get all users (Admin) |
| `/api/users/{id}` | GET | Get user by ID |
| `/api/users/{id}` | PATCH | Update user profile |
| `/api/users/{id}/password` | PATCH | Change password |
| `/api/gdpr/export` | POST | Request data export |

#### Hotels Service (Port 8082)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/hotels/search` | GET | Search hotels with filters |
| `/api/hotels/{id}` | GET | Get hotel details |
| `/api/hotels` | POST | Submit new hotel |
| `/api/hotels/{id}` | PATCH | Update hotel |
| `/api/hotels/mine` | GET | Get own hotels |
| `/api/rooms/{id}` | GET | Get room details |
| `/api/rooms` | POST | Add room to hotel |
| `/api/admin/hotels/pending` | GET | Get pending hotels |
| `/api/admin/hotels/{id}/approve` | POST | Approve hotel |
| `/api/admin/hotels/{id}/reject` | POST | Reject hotel |

#### Reservations Service (Port 8083)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/reservations` | POST | Create reservation |
| `/api/reservations/{id}` | GET | Get reservation details |
| `/api/reservations/mine` | GET | Get own reservations |
| `/api/reservations/{id}/cancel` | POST | Request cancellation |
| `/api/admin/reservations` | GET | Get all reservations |
| `/api/admin/reservations/cancellations` | GET | Get pending cancellations |
| `/api/admin/reservations/{id}/approve-cancel` | POST | Approve cancellation |
| `/api/admin/reservations/{id}/reject-cancel` | POST | Reject cancellation |

#### Payments Service (Port 8084)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/payments/create-intent` | POST | Create payment intent |
| `/api/payments/confirm` | POST | Confirm payment |
| `/api/payments/{id}` | GET | Get payment details |
| `/api/payments/reservation/{id}` | GET | Get payment by reservation |
| `/api/webhooks/stripe` | POST | Stripe webhook handler |

### Swagger UI Access

| Service | Swagger URL |
|---------|-------------|
| Users | https://hotel-booking-users-api-csbghtd2f9cph7g5.northeurope-01.azurewebsites.net/swagger/index.html |
| Hotels | https://hotel-booking-hotels-api-evhhefafhhbrgrbs.northeurope-01.azurewebsites.net/swagger/index.html |
| Reservations | https://hotel-booking-reservations-api-dwfzh9bydth3fke0.northeurope-01.azurewebsites.net/swagger/index.html |
| Payments | https://hotel-booking-payments-api-gch8e7fyeqenfje8.northeurope-01.azurewebsites.net/swagger/index.html |

## Requirements Checklist

### Minimum Requirements (Grade 5)

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1 | Structured API documentation | ✅ | Swagger UI with organized endpoints |
| 2 | OpenAPI/Swagger specification | ✅ | Swashbuckle generates valid OpenAPI 3.0 |
| 3 | Endpoint examples (requests/responses) | ✅ | XML comments with sample JSON |
| 4 | HTTP status codes documented | ✅ | ProducesResponseType on all methods |
| 5 | Data model descriptions | ✅ | DTO classes with XML documentation |
| 6 | Getting started guide | ✅ | docs/03-user-guide/index.md |
| 7 | Published in accessible format | ✅ | Swagger UI on Azure URLs |
| 8 | Documentation strategy described | ✅ | This document + code annotations |

### Maximum Requirements (Grade 10)

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 1 | Comprehensive docs (guides, tutorials, versioning) | ⚠️ Partial | Getting started exists, no versioning docs |
| 2 | The API documentation must thoroughly describe advanced topics | ⚠️ Partial | Request flows, auth documented |
| 3 | Detailed diagrams (sequence, component) | ✅ | Architecture diagrams in reference/ |
| 4 | API linting/validation | ❌ | No Spectral or schema validation |

## Known Limitations

| Limitation | Impact | Potential Solution |
|------------|--------|-------------------|
| No API versioning | Breaking changes affect all clients | Implement URL or header-based versioning |
| No rate limiting documentation | Clients unaware of limits | Document rate limits when implemented |
| Internal endpoints visible | Could confuse external developers | Separate internal/external Swagger docs |

## References

- [Swashbuckle.AspNetCore](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)
- [OpenAPI Specification](https://swagger.io/specification/)
- Swagger configuration: `Program.cs` in each service
- Controller documentation: `Api/Controllers/` in each service
