# Hotel Booking Platform - Testing Documentation

## Overview

This document provides comprehensive testing documentation for the Hotel Booking Platform, a microservices-based application consisting of four independent services: Users, Hotels, Reservations, and Payments. All services implement a robust automated testing strategy following industry best practices and FIRST principles.

## Testing Strategy

The testing strategy is built on a solid foundation of automated tests that verify both individual component behavior and cross-component interactions. Each microservice maintains its own comprehensive test suite that validates the core business logic, API endpoints, authentication/authorization mechanisms, and data access layers.


## Test Types Implemented

### 1. Unit Tests

Unit tests verify individual components in isolation using mocked dependencies. All tests follow the Arrange-Act-Assert (AAA) pattern and ensure that business logic is tested independently of infrastructure concerns.

**Coverage Areas:**
- **Domain Layer**: Entity behavior, validation rules, state transitions, and business logic
- **Application Layer**: Command and query handlers, DTOs, and application services
- **Infrastructure Layer**: Repository implementations, authentication services, authorization logic, and HTTP client handlers

**Characteristics:**
- Dependencies mocked using Moq framework
- No external dependencies (databases, APIs, file systems)
- In-memory databases with unique identifiers per test for complete isolation
- Fast execution (unit test suites complete in under 1 second)

### 2. Integration Tests

Integration tests validate interactions between multiple components and verify that the system works correctly when components are combined.

**Coverage Areas:**
- **API Controllers**: Full request/response cycle testing using `WebApplicationFactory`
- **Database Operations**: Repository tests using EF Core In-Memory provider
- **Authentication Pipeline**: JWT and API key authentication flow verification
- **Authorization**: Role-based and resource-based access control validation
- **Configuration**: Dependency injection and service registration verification

**Characteristics:**
- Full ASP.NET Core integration with `WebApplicationFactory`
- In-memory database per test class (isolated via unique database names)
- Custom test authentication handlers for role/permission testing
- HTTP request/response validation with real middleware pipeline

### 3. Contract Tests

The Users Service implements contract tests to validate service-to-service communication protocols.

**Coverage:**
- Communication contracts with Reservations and Payments services
- HTTP request/response format validation
- Service integration point verification using WireMock.Net

## Code Coverage

All services exceed the required 70% code coverage threshold for both line and branch coverage. The automated test suites provide comprehensive coverage of business logic, authentication, authorization, and data access layers.

### Coverage Approach

Coverage is collected using **Coverlet**, a cross-platform code coverage library for .NET, with reports generated in Cobertura XML format and visualized using **ReportGenerator** HTML reports.

### Coverage Exclusions

To maintain meaningful coverage metrics, certain categories of code are excluded from coverage calculations. These exclusions are implemented through two mechanisms:

#### 1. Attribute-Based Exclusions (`[ExcludeFromCodeCoverage]`)

Files marked with this attribute contain framework infrastructure code with no testable business logic:

- **Middleware Components**: `GlobalExceptionHandler`, `CorrelationIdMiddleware` - framework-level request processing
- **Swagger Filters**: `ExampleSchemaFilter` - API documentation generation
- **Dependency Injection**: `DependencyInjection.cs` - service registration and composition root
- **Database Migrations**: EF Core generated migration files and model snapshots
- **Internal API Controllers**: Service-to-service communication endpoints with simple data transfer operations

#### 2. Project-Level Exclusions (Test .csproj Configuration)

Test projects exclude additional patterns through Coverlet configuration:

```xml
<Exclude>
  [*]*.Migrations.*,
  [*]*ModelSnapshot,
  [*]*DbContextFactory,
  [*]*DataSeeder*,
  [*]*.Configurations.*
</Exclude>

<ExcludeByAttribute>
  Obsolete,
  GeneratedCode,
  CompilerGenerated,
  ExcludeFromCodeCoverage
</ExcludeByAttribute>
```

**Rationale for Exclusions:**
- **Migrations**: Auto-generated EF Core database migration code
- **DbContext Factories**: Design-time scaffolding used only by EF Core tooling
- **Data Seeders**: Development/testing data population utilities
- **Program.cs**: Application entry point and framework bootstrap code
- **Configuration Classes**: Simple DTOs for framework configuration

These exclusions represent infrastructure and framework code that either cannot be meaningfully tested or provide no business value when tested.

## FIRST Principles Compliance

All test suites strictly adhere to FIRST principles to ensure reliability, maintainability, and effectiveness:

### Fast
- Unit test suites execute in under 1 second per service
- Integration tests complete in 2-5 seconds per service
- In-memory databases eliminate disk I/O overhead
- Mocked external dependencies prevent network latency
- Parallel test execution enabled where appropriate

### Independent
- Each test creates isolated dependencies with unique identifiers (`Guid.NewGuid()`)
- Fresh mock instances per test prevent shared state
- In-memory databases reset between tests
- No test execution order dependencies
- Tests that modify environment variables restore original values in cleanup

### Repeatable
- Deterministic test data and behavior
- Fixed timestamps with appropriate tolerance for time-based assertions
- All external services mocked or stubbed
- In-memory databases provide consistent starting state
- Tests pass consistently across multiple runs and environments

### Self-Validating
- Clear pass/fail criteria using FluentAssertions DSL
- Descriptive assertion messages explain failures
- No manual verification or inspection required
- Automated verification of all expected outcomes

### Timely
- Tests written alongside implementation code
- Test-driven development approach for domain logic
- Edge cases covered as they are discovered
- Regression tests added for identified bugs

## Test Organization and Naming
### Naming Conventions

**Test Files**: `{ComponentName}Tests.cs`

**Test Methods**: `{MethodName}_{Scenario}_{ExpectedBehavior}`

Examples:
- `GetByIdAsync_WithExistingId_ShouldReturnReservation`
- `Create_ShouldReturnForbidden_WhenCreatingForDifferentOwner`
- `Login_WithValidCredentials_ShouldReturnToken`
- `Update_ShouldAllowNullOptionalValues`

## Running Tests

### Quick Test Run

Execute all tests for a specific service with ```dotnet test``` command

### Run Tests with Coverage Report

Each service includes a PowerShell script that runs tests and generates an HTML coverage report:

```powershell
.\run-tests-with-coverage.ps1
```

This script performs the following operations:
1. Cleans previous coverage data from the coverage directory
2. Executes all tests with Coverlet coverage collection
3. Generates Cobertura XML coverage report
4. Installs ReportGenerator tool globally (if not already installed)
5. Generates interactive HTML coverage report
6. Opens the report in the default browser

The HTML report provides detailed coverage visualization including:
- Line-by-line coverage highlighting
- Branch coverage visualization
- Coverage metrics by namespace and class
- Historical coverage trends (when run multiple times)

### Coverage Report Locations

After running the coverage script, HTML reports are available at:

- **Users Service**: `hotel-booking-users-service/CoverageReport/Report/index.html`
- **Hotels Service**: `hotel-booking-hotels-service/Tests/coverage/report/index.html`
- **Reservations Service**: `hotel-booking-reservations-service/Tests/CoverageReport/index.html`
- **Payments Service**: `hotel-booking-payments-service/Tests/CoverageReport/index.html`


## Tools and Frameworks

| Tool | Purpose |
|------|---------|
| **xUnit** | Primary testing framework providing test runner and assertion infrastructure |
| **FluentAssertions** | Expressive assertion library with readable test syntax |
| **Moq** | Mocking framework for creating test doubles |
| **Coverlet** | Cross-platform code coverage collection library |
| **ReportGenerator** | HTML coverage report generation and visualization |
| **Microsoft.AspNetCore.Mvc.Testing** | Integration testing framework with `WebApplicationFactory` |
| **Microsoft.EntityFrameworkCore.InMemory** | In-memory database provider for isolated testing |
| **WireMock.Net** | HTTP service mocking for contract tests (Users Service) |

## Test Isolation and Database Strategy

All services use EF Core In-Memory database provider for test isolation:

- Each test class or test uses a unique database name (typically `Guid.NewGuid().ToString()`)
- No shared state between tests
- Database automatically disposed after test completion
- No need for transaction rollback or cleanup

**Known Limitations of In-Memory Provider:**
- `ExecuteUpdateAsync` and `ExecuteDeleteAsync` not supported (gracefully handled in tests)
- No support for raw SQL queries
- Transactions are no-ops (not needed for in-memory scenarios)

## Authentication and Authorization Testing

### JWT Authentication Testing

Integration tests verify JWT authentication flows:
- Test users created and authenticated via API endpoints
- Bearer tokens extracted and added to HTTP request headers
- Token validation uses production configuration
- Role and permission claims validated

### API Key Authentication Testing

Service-to-service authentication tested using:
- `X-API-Key` header for internal endpoints
- Test API keys configured in `TestWebApplicationFactory`
- Service principal claim validation

### Authorization Testing

Resource-based authorization tested with scenarios:
- Admin access to all resources
- User access to owned resources only
- Service account access patterns
- Unauthorized access rejection

## Known Testing Gaps

While all services exceed the 70% coverage requirement, some areas have intentionally limited test coverage:

### Infrastructure Edge Cases

Complex error handling paths in infrastructure components (HTTP client handlers, authentication handlers) that are difficult to reproduce in unit tests. These scenarios are typically handled by framework exception handling and represent defensive programming rather than core business logic.



### Complex LINQ Query Branches

Certain database query edge cases that require specific data states challenging to set up in test environments. Core query functionality is thoroughly tested through integration tests.

**Justification**: These gaps represent a small portion of the codebase (well within the acceptable 30% untested code threshold) and are in infrastructure/framework integration code rather than business logic. All core business logic maintains near-100% coverage.



## Test Statistics Summary

Across all four microservices:

- **Total Tests**: 790+ tests
- **Average Execution Time**: 1-5 seconds per service
- **Line Coverage**: >70% for all services 
- **Branch Coverage**: >70% for all services
- **Test Frameworks**: xUnit (all services)
- **Assertion Libraries**: FluentAssertions (all services)
- **Mocking**: Moq (all services)


